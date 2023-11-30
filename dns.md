
 ISUCON13で登場したDNS周りの知見をメモとして残しておく

DNS参考実装

[https://gist.github.com/ryotarai/2938ac139c23417da222c1a387415836](https://gist.github.com/ryotarai/2938ac139c23417da222c1a387415836)

[https://github.com/icchy/isucon13/blob/main/isudns/main.go](https://github.com/icchy/isucon13/blob/main/isudns/main.go)

  

if out, err := exec.CommandContext(CleanContext(c.Request().Context()),

"ssh", "192.168.0.13",

"sudo", "-u", "pdns",

"sh", "-c",

fmt.Sprintf(`"echo '%s 60 A %s' >> /var/lib/powerdns/zones/u.isucon.dev.zone && pdns_control bind-reload-now u.isucon.dev"`, req.Name, powerDNSSubdomainAddress),

).CombinedOutput(); err != nil {

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ5ODExOTk2N119
-->