# General mythtv tips



### Frontend connect errors on later versions of mariadb/mysql due to default encryption enabled on clients.

<code>
QMYSL: Unable to connect
Database error was:
TLS/SSL error : SSL is required, but the server does not support it
</code>

Test starting the frontend with :
<code>
export MARIADB_TLS_DISABLE_PEER_VERIFICATION=1
</code>

If that works add it to your environment or a mythfrontend startup script.