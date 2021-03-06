---
description: 版本：10
---

# 19.3. 連線與認證

## 19.3.1. 連線設定

`listen_addresses` \(`string`\)

指定伺服器監聽用戶端應用程序連線的 TCP/IP 位址。該值採用逗號分隔的主機名稱或數字 IP 位址列表的形式。特殊項目「\*」對應於所有可用的 IP。項目 0.0.0.0 允許監聽所有 IPv4 位址，還有「::」允許監聽所有 IPv6 位址。如果列表為空，則伺服器根本不監聽任何 IP 接口，在這種情況下，就只能使用 Unix-domain socket 來連接它。預設值是 localhost，它只允許進行本地 TCP/IP loopback 連線。儘管用戶端身份驗證（[第 20 章](../client-authentication/)）允許對誰可以存取伺服器進行細維的控制，但 listen\_addresses 控制哪些 IP 接受連線嘗試，這有助於防止在不安全的網路接口上重複發出惡意的連線請求。此參數只能在伺服器啟動時設定。

`port` \(`integer`\)

伺服器監聽的 TCP 連接埠；預設是 5432。請注意，相同的連接埠號號用於伺服器監聽的所有 IP 地址。此參數只能在伺服器啟動時設定。

`max_connections` \(`integer`\)

決定資料庫伺服器的最大同時連線數。預設值通常為 100 個連線，但如果您的核心設定不支援它（在 initdb 期間確定），則可能會更少。該參數只能在伺服器啟動時設定。

運行備用伺服器時，必須將此參數設定為與主服務器上相同或更高的值。 否則，查詢將不被允許在備用伺服器中使用。

`superuser_reserved_connections` \(`integer`\)

決定為 PostgreSQL 超級使用者連線保留的連線「插槽」的數量。最多 max\_connections 連線可以同時活動。當活動同時連線的數量為 max\_connections 減去 superuser\_reserved\_connections 以上時，新連線將僅接受超級使用者，並且不會接受新的複寫作業連線。

預設值是三個連線。該值必須小於 max\_connections 的值。此參數只能在伺服器啟動時設定。

`unix_socket_directories` \(`string`\)

指定伺服器要監聽來自用戶端應用程序以 Unix-domain socket 連線的目錄。列出由逗號分隔的多個目錄可以建立多個 socket。項目之間的空白會被忽略；如果您需要在名稱中包含空格或逗號，請用雙引號括住目錄名稱。空值表示不監聽任何 Unix-domain socket，在這種情況下，只有 TCP/IP 協定可用於連線到服務器。預設值通常是 /tmp，但可以在編譯時變更。此參數只能在伺服器啟動時設定。

除了名為 .s.PGSQL.nnnn 的 socket 檔案本身之外，其中 nnnn 是伺服器的連接埠號號，將在每個 unix\_socket\_directories 目錄中建立一個名為 .s.PGSQL.nnnn.lock 的普通檔案。這兩個檔案都不應該手動刪除。

這個參數與 Windows 無關，它沒有 Unix-domain socket。

`unix_socket_group` \(`string`\)

設定 Unix-domain socket 的群組。（socket 的使用者始終是啟動伺服器的使用者。）結合參數 unix\_socket\_permissions，可以將其用作為 Unix-domain socket 的附加存取控制機制。預設情況下，這是空字符串，它使用服務器用戶的預設群組。此參數只能在伺服器啟動時設定。

這個參數與 Windows 無關，它沒有 Unix-domain socket。

`unix_socket_permissions` \(`integer`\)

設定 Unix-domain socket 的存取權限。Unix-domain socket 使用一般的 Unix 檔案系統權限設定。期望的參數值是以 chmod 和 umask 系統呼叫可接受的格式指定數字模式。（要使用習慣的八進制格式，數字必須以 0（零）開頭。）

預設權限是 0777，意味著任何人都可以進行連線。合理的選擇是 0770（僅使用者和其群組，另請參閱 unix\_socket\_group）和 0700（僅使用者本身）。（請注意，對於Unix-domain socket，只有寫入權限很重要，所以設定還是撤消讀取或執行權限都沒有意義。）

這種存取控制機制獨立於[第 20 章](../client-authentication/)中所描述的機制。

此參數只能在伺服器啟動時設定。

此參數在某些系統上無關緊要，特別是從 Solaris 10 開始的 Solaris，會完全忽略權限許可。在那裡，透過將 unix\_socket\_directories 指向具有僅限於所需的搜尋權限的目錄，就可以實現類似的效果。這個參數與 Windows 也是無關的，它沒有 Unix-domain socket。

`bonjour` \(`boolean`\)

透過 Bonjour 啟用伺服器存在的廣播。預設是關閉的。此參數只能在伺服器啟動時設定。

`bonjour_name` \(`string`\)

指定 Bonjour 的服務名稱。如果此參數設定為空字串''（這是預設值），則使用電腦名稱。 如果伺服器未使用 Bonjour 支援進行編譯，則此參數將被忽略。此參數只能在伺服器啟動時設定。

`tcp_keepalives_idle` \(`integer`\)

指定 TCP 在發送 Keepalive 訊息給用戶端之後保持連線的秒數。值為 0 時使用系統預設值。此參數僅在支援 TCP\_KEEPIDLE 或等效網路選項的系統上以及在 Windows 上受到支援；在其他系統上，它必須是零。在透過 Unix-domain socket 的連線中，該參數將被忽略並始終為零。

#### 注意

在 Windows 上，值為 0 會將此參數設定為2小時，因為 Windows 不提供讀取系統預設值的方法。

`tcp_keepalives_interval` \(`integer`\)

指定用戶端未回應的 TCP 保持活動訊息應重新傳輸的秒數。值為 0 時使用系統預設值。此參數僅在支援 TCP\_KEEPINTVL 或等效網路選項的系統上以及在 Windows 上受到支援；在其他系統上，它必須是零。在透過 Unix-domain socket 的連線中，此參數將被忽略並始終為零。

#### 注意

在 Windows 上，值為 0 會將此參數設定為 1 秒，因為 Windows 不提供讀取系統預設值的方法。

`tcp_keepalives_count` \(`integer`\)

指定在伺服器連線到用戶端之前可能已經失去的 TCP 保持連線的數量。值為 0 時使用系統預設值。此參數僅在支援 TCP\_KEEPCNT 或等效網路選項的系統上受到支持；在其他系統上，它必須是零。在透過 Unix-domain socket 的連線中，此參數將被忽略並始終為零。

#### 注意

此參數在 Windows 上不支援，並且必須為零。

## 19.3.2. Security and Authentication

`authentication_timeout` \(`integer`\)

Maximum time to complete client authentication, in seconds. If a would-be client has not completed the authentication protocol in this much time, the server closes the connection. This prevents hung clients from occupying a connection indefinitely. The default is one minute \(`1m`\). This parameter can only be set in the `postgresql.conf` file or on the server command line.

`ssl` \(`boolean`\)

Enables SSL connections. Please read [Section 18.9](https://www.postgresql.org/docs/10/static/ssl-tcp.html) before using this. This parameter can only be set in the `postgresql.conf` file or on the server command line. The default is `off`.

`ssl_ca_file` \(`string`\)

Specifies the name of the file containing the SSL server certificate authority \(CA\). Relative paths are relative to the data directory. This parameter can only be set in the `postgresql.conf` file or on the server command line. The default is empty, meaning no CA file is loaded, and client certificate verification is not performed.

In previous releases of PostgreSQL, the name of this file was hard-coded as `root.crt`.

`ssl_cert_file` \(`string`\)

Specifies the name of the file containing the SSL server certificate. Relative paths are relative to the data directory. This parameter can only be set in the `postgresql.conf` file or on the server command line. The default is `server.crt`.

`ssl_crl_file` \(`string`\)

Specifies the name of the file containing the SSL server certificate revocation list \(CRL\). Relative paths are relative to the data directory. This parameter can only be set in the `postgresql.conf` file or on the server command line. The default is empty, meaning no CRL file is loaded.

In previous releases of PostgreSQL, the name of this file was hard-coded as `root.crl`.

`ssl_key_file` \(`string`\)

Specifies the name of the file containing the SSL server private key. Relative paths are relative to the data directory. This parameter can only be set in the `postgresql.conf` file or on the server command line. The default is `server.key`.

`ssl_ciphers` \(`string`\)

Specifies a list of SSL cipher suites that are allowed to be used on secure connections. See the ciphers manual page in the OpenSSL package for the syntax of this setting and a list of supported values. This parameter can only be set in the `postgresql.conf` file or on the server command line. The default value is `HIGH:MEDIUM:+3DES:!aNULL`. The default is usually a reasonable choice unless you have specific security requirements.

Explanation of the default value:`HIGH`

Cipher suites that use ciphers from `HIGH` group \(e.g., AES, Camellia, 3DES\)`MEDIUM`

Cipher suites that use ciphers from `MEDIUM` group \(e.g., RC4, SEED\)`+3DES`

The OpenSSL default order for `HIGH` is problematic because it orders 3DES higher than AES128. This is wrong because 3DES offers less security than AES128, and it is also much slower. `+3DES` reorders it after all other `HIGH` and `MEDIUM` ciphers.`!aNULL`

Disables anonymous cipher suites that do no authentication. Such cipher suites are vulnerable to man-in-the-middle attacks and therefore should not be used.

Available cipher suite details will vary across OpenSSL versions. Use the command `openssl ciphers -v 'HIGH:MEDIUM:+3DES:!aNULL'` to see actual details for the currently installed OpenSSL version. Note that this list is filtered at run time based on the server key type.

`ssl_prefer_server_ciphers` \(`boolean`\)

Specifies whether to use the server's SSL cipher preferences, rather than the client's. This parameter can only be set in the `postgresql.conf` file or on the server command line. The default is `true`.

Older PostgreSQL versions do not have this setting and always use the client's preferences. This setting is mainly for backward compatibility with those versions. Using the server's preferences is usually better because it is more likely that the server is appropriately configured.

`ssl_ecdh_curve` \(`string`\)

Specifies the name of the curve to use in ECDH key exchange. It needs to be supported by all clients that connect. It does not need to be the same curve used by the server's Elliptic Curve key. This parameter can only be set in the `postgresql.conf` file or on the server command line. The default is `prime256v1`.

OpenSSL names for the most common curves are: `prime256v1` \(NIST P-256\), `secp384r1` \(NIST P-384\), `secp521r1` \(NIST P-521\). The full list of available curves can be shown with the command `openssl ecparam -list_curves`. Not all of them are usable in TLS though.

`password_encryption` \(`enum`\)

When a password is specified in [CREATE ROLE](https://www.postgresql.org/docs/10/static/sql-createrole.html) or [ALTER ROLE](https://www.postgresql.org/docs/10/static/sql-alterrole.html), this parameter determines the algorithm to use to encrypt the password. The default value is `md5`, which stores the password as an MD5 hash \(`on` is also accepted, as alias for `md5`\). Setting this parameter to `scram-sha-256` will encrypt the password with SCRAM-SHA-256.

Note that older clients might lack support for the SCRAM authentication mechanism, and hence not work with passwords encrypted with SCRAM-SHA-256. See [Section 20.3.2](https://www.postgresql.org/docs/10/static/auth-methods.html#AUTH-PASSWORD) for more details.

`ssl_dh_params_file` \(`string`\)

Specifies the name of the file containing Diffie-Hellman parameters used for so-called ephemeral DH family of SSL ciphers. The default is empty, in which case compiled-in default DH parameters used. Using custom DH parameters reduces the exposure if an attacker manages to crack the well-known compiled-in DH parameters. You can create your own DH parameters file with the command `openssl dhparam -out dhparams.pem 2048`.

This parameter can only be set in the `postgresql.conf` file or on the server command line.

`krb_server_keyfile` \(`string`\)

Sets the location of the Kerberos server key file. See [Section 20.3.3](https://www.postgresql.org/docs/10/static/auth-methods.html#GSSAPI-AUTH) for details. This parameter can only be set in the `postgresql.conf` file or on the server command line.

`krb_caseins_users` \(`boolean`\)

Sets whether GSSAPI user names should be treated case-insensitively. The default is `off` \(case sensitive\). This parameter can only be set in the `postgresql.conf` file or on the server command line.

`db_user_namespace` \(`boolean`\)

This parameter enables per-database user names. It is off by default. This parameter can only be set in the `postgresql.conf` file or on the server command line.

If this is on, you should create users as _`username@dbname`_. When _`username`_ is passed by a connecting client, `@` and the database name are appended to the user name and that database-specific user name is looked up by the server. Note that when you create users with names containing `@` within the SQL environment, you will need to quote the user name.

With this parameter enabled, you can still create ordinary global users. Simply append `@` when specifying the user name in the client, e.g. `joe@`. The `@` will be stripped off before the user name is looked up by the server.

`db_user_namespace` causes the client's and server's user name representation to differ. Authentication checks are always done with the server's user name so authentication methods must be configured for the server's user name, not the client's. Because `md5` uses the user name as salt on both the client and server, `md5` cannot be used with `db_user_namespace`.

#### Note

This feature is intended as a temporary measure until a complete solution is found. At that time, this option will be removed.

