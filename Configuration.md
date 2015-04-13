# Introduction #

Squirtle is configured through a YAML-structured configuration file. Defaults are used if the item does not exist within the configuration file.

# Details #

| **Item** | **Default Value** | **Description**|
|:---------|:------------------|:|
| domain |  DOMAIN | Windows domain to use |
| server | SERVER | Server name to use |
| dns\_domain | example.com | DNS domain suffix |
| serverstring | Microsoft-IIS/6.0 | Server agent string |
| msflib | Current Working Directory | Location of Metasploit 3 |
| port | 8080 | Webserver listening port |
| address | 0.0.0.0 | Webserver listening address |
| nonce | 1122334455667788 | Static nonce |
| output-file | ntlmhashes.txt | File to write NT/LM hashes to for cracking |
| dbfile | squirtle.db | SQLite database file to use |
| authenticate | password | Authentication password (not implemented yet) |
| timeout | 5000 | How many milliseconds between keepalive messages. Clients who do not respond after missing 5 keepalives will be purged |
| indexfile | squirtle.html | HTML file to load as the default index page |
| jsfile | squirtle.js | Javascript file to load into the default index page |