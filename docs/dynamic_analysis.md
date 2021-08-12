## **Objective**

The aim of this section is to perform dynamic analysis using DAST tools on DVNA.

About DAST

- Dynamic analysis is done while an application is running. 
- DAST tools include OWASP Zed Attack Proxy (ZAP) and W3AF.


### **OWASP ZAP**

Implementing ZAP analysis with docker is simpler and faster than manual installation. I followed this [documentation](https://www.zaproxy.org/docs/docker/about/). First, pull the ZAP image from docker hub.
```bash
sudo docker pull owasp/zap2docker-stable
```

TRIED THIS! It Didnt Work!
Create a docker network for both application container and zap container to run in.
```bash
sudo docker network create zapnet
```

```bash
sudo docker run -u zap -td --name owasp-zap --net zapnet -p 8090:8090 owasp/zap2docker-stable zap.sh -daemon -port 8090 -host 0.0.0.0 -config api.disablekey=true

sudo docker exec owasp-zap zap-cli open-url http://192.168.56.102:9090

sudo docker exec owasp-zap zap-cli active-scan http://192.168.56.102:9090
```



FINAL SOLUTION
The baseline-scan script is intended to be ideal to run in a CI/CD environment, even against production sites.
Docker flags used

- --rm, remove container after completion
- -d, run as a background job
- -u <user>, specify user to run container as
- -v 'host dir':'container dir', mount volumes

Zap CLI flags used

- -t 'target', specify target to scan
- -r 'file.html', generate an HTML output report
- -l level, minimum level to show: PASS, IGNORE, INFO, WARN or FAIL.

```bash
sudo docker run --rm -td -u zap --name owasp-zap -v ~/:/zap/wrk/ owasp/zap2docker-stable zap-baseline.py -t http://192.168.56.102:9090 -r owasp-zap-report.html -l PASS
```

To run a fullscan script, run the following command
```bash
sudo docker run --rm -td -u zap --name owasp-zap -v ~/:/zap/wrk/ owasp/zap2docker-stable zap-full-scan.py -t http://192.168.56.102:9090 -r owasp-zap-report.html -l PASS
```

The report `owasp-zap-report.html` will be generated on successful completion. It will be located in the users home directory.
**Note:** To open .html file in browser from terminal, type `open owasp-report.html` from host terminal.
