# try-finally 보다 try-with-resource

자바 라이브러리에는 close 메소드를 호출해 직접 닫아줘야 하는 자원이 많다.

자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 한다. 자원 중 상당수가 안전망으로 finalizer을 활용하고 있는데 finalizer는 그렇게 믿을만하지 못하다.

심지어 자원을 try finally를 사용하게 되어서 닫을 때 하나가 아닌 2개 이상일 경우에는 프론트의 콜백지옥 마냥 계속해서 지저분하게 코드가 완성되게 된다.

```
  public void test(String test1, String test2) {
    InputStream in = new FileInputStream(test1);
    try {
      OutputStream out = new FileOutputStream(test2);
      try {
        ...중략
      } finally {
        out.close();
      }
    } finally {
      in.close();
    }
  }
```

해당문제를 try-with-resources를 통해서 try() 안에 선언할 자원들을 넣어줌으로써 try ()안에서 AutoCloseable이 작동해 자동으로 닫아주는 시스템이 완성되는 것이다.

밑에는 프로젝트 때문에 실제로 작성한 코드다.

```
  public class DBSetup {
    public static void initDB(Plugin plugin, DataSource dataSource) throws SQLException,IOException {
        String setup = "";

        try(InputStream inputStream = DBSetup.class.getClassLoader().getResourceAsStream("setup.sql")) {
            setup = new BufferedReader(new InputStreamReader(inputStream)).lines().collect(Collectors.joining("\n"));
        } catch (IOException e) {
            plugin.getLogger().log(Level.SEVERE, "DB 셋업 파일을 인식할 수 없습니다.", e);
        }

        String[] queries = setup.split(";");
        try(Connection conn = dataSource.getConnection()) {
            conn.setAutoCommit(false);
            for(String query: queries) {
                if(query.isEmpty()) continue;
                try (PreparedStatement stmt = conn.prepareStatement(query)) {
                    plugin.getLogger().info(query);
                    stmt.execute();
                }
            }
            conn.commit();
        } catch(SQLException e) {
            plugin.getLogger().log(Level.SEVERE, "쿼리 인식 과정에 문제 발생", e);
        }

        plugin.getLogger().info("DB 테이블 세팅이 완료되었습니다.");
    }
}
```

일반적인 try-finally마냥 try-with-resources도 catch절을 쓸 수 있다.
