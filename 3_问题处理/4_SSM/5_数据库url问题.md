
问题情况 : javax.net.ssl.SSLException: closing inbound before receiving peer‘s close_notify异常


解决办法 : 在数据库连接配置文件中，配置连接数据库的url时，加上useSSL=false。
