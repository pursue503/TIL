# logstash Simple Test


```bash
input {
  jdbc {
    jdbc_driver_library => "mysql-connector-java-8.0.28.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://DB_URL/db_schema"
    jdbc_user => "user_id"
    jdbc_password => "pw"

    statement => "select * from item" # 실행할 쿼리문

    jdbc_pool_timeout => 10 #jdbc 접속 TimeOut 설정
    jdbc_paging_enabled => true 
    jdbc_page_size => 10000

    schedule => "* * * * *"  # crontab 표기법의 스케쥴 설정
  }
}

output {
  stdout {
    codec => rubydebug	
  }
    elasticsearch {
      hosts => "127.0.0.1:9300"
      index => "item"
      document_id=>"%{id}" 
#      user => "el"
#      password => "el"
    }
}
```