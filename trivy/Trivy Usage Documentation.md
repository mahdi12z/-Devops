### 1. **Installing Trivy**

To install Trivy on Linux systems, first download the `.deb` file and install it:
```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.41.0/trivy_0.41.0_Linux-64bit.deb
sudo dpkg -i trivy_0.41.0_Linux-64bit.deb
```

### 2. **Scanning Kubernetes Config (nginx-deployment.yaml)**

To scan Kubernetes resources using Trivy, first clear the AWS-specific policies cache to avoid using AWS-specific rules:
```bash
rm -rf ~/.cache/trivy/policy/content/policies/cloud/policies/aws
```

Then, scan the Kubernetes configuration file:
```bash
trivy config ./nginx-deployment.yaml
```

Example `nginx-deployment.yaml` file:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```
3. **Scanning Docker Image** :To scan a Docker image using Trivy, run the following command:

```
 docker pull nginx:alpine
```

```bash
trivy image nginx:alpine
```
This command scans the `nginx:latest` image and Trivy will display the security issues and vulnerabilities found within that image.

---
### Scanning Git Repositories for Misconfigurations Using Trivy

Trivy allows you to scan remote Git repositories to detect security misconfigurations and vulnerabilities. If you're only interested in **configuration issues**—such as insecure Dockerfiles, Kubernetes manifests, or Terraform scripts—you can limit the scan to just that.

#### Command:
```bash
trivy repo --vuln-type config https://github.com/knqyf263/trivy-ci-test

```



---
### **Trivy File System Scan:**

Trivy can scan the file system to identify vulnerabilities, misconfigurations, and sensitive data. Below are two commonly used commands for scanning the file system.
```bash
trivy fs /opt trivy fs --security-checks vuln,config,secret /opt

rm -rf ~/.cache/trivy/policy/content/policies/cloud/policies/aws
```

**Explanation:**

- This command performs a more comprehensive scan on the `/opt` directory, covering:
    
    1. **Vulnerabilities** (`vuln`): Scans for security vulnerabilities in the packages and libraries.
        
    2. **Configurations** (`config`): Analyzes the configurations to identify security misconfigurations (e.g., unsafe configurations in Kubernetes, Docker, etc.).
        
    3. **Secrets** (`secret`): Searches for sensitive data such as passwords, API keys, tokens, and other secrets within the file system.
        
- This scan provides a broader security assessment, and it is recommended if you need to evaluate not only vulnerabilities but also security misconfigurations and sensitive information within the file system.

-----
## Trivy Scan Output and Custom Template Example
### 1 - **JSON output**
To generate a JSON report of vulnerabilities, misconfigurations, and secrets for a specific directory or filesystem, use the following command:

```
trivy fs --security-checks vuln,config,secret /home/aress/ -f json -o result.json
```

### 2-**Custom template.**
To generate a custom formatted report (in HTML) with detailed information about vulnerabilities and misconfigurations, follow these steps:

1. **Create the Custom HTML Template (`html.tpl`)**
   First, create the custom HTML template (`html.tpl`) by editing it using a text editor like `vi`:

```
vi html.tpl
```

```
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
{{- if . }}
    <style>
      * {
        font-family: Arial, Helvetica, sans-serif;
      }
      h1 {
        text-align: center;
      }
      .group-header th {
        font-size: 200%;
      }
      .sub-header th {
        font-size: 150%;
      }
      table, th, td {
        border: 1px solid black;
        border-collapse: collapse;
        white-space: nowrap;
        padding: .3em;
      }
      table {
        margin: 0 auto;
      }
      .severity {
        text-align: center;
        font-weight: bold;
        color: #fafafa;
      }
      .severity-LOW .severity { background-color: #5fbb31; }
      .severity-MEDIUM .severity { background-color: #e9c600; }
      .severity-HIGH .severity { background-color: #ff8800; }
      .severity-CRITICAL .severity { background-color: #e40000; }
      .severity-UNKNOWN .severity { background-color: #747474; }
      .severity-LOW { background-color: #5fbb3160; }
      .severity-MEDIUM { background-color: #e9c60060; }
      .severity-HIGH { background-color: #ff880060; }
      .severity-CRITICAL { background-color: #e4000060; }
      .severity-UNKNOWN { background-color: #74747460; }
      table tr td:first-of-type {
        font-weight: bold;
      }
      .links a,
      .links[data-more-links=on] a {
        display: block;
      }
      .links[data-more-links=off] a:nth-of-type(1n+5) {
        display: none;
      }
      a.toggle-more-links { cursor: pointer; }
    </style>
    <title>{{- escapeXML ( index . 0 ).Target }} - Trivy Report - {{ now }} </title>
    <script>
      window.onload = function() {
        document.querySelectorAll('td.links').forEach(function(linkCell) {
          var links = [].concat.apply([], linkCell.querySelectorAll('a'));
          [].sort.apply(links, function(a, b) {
            return a.href > b.href ? 1 : -1;
          });
          links.forEach(function(link, idx) {
            if (links.length > 3 && 3 === idx) {
              var toggleLink = document.createElement('a');
              toggleLink.innerText = "Toggle more links";
              toggleLink.href = "#toggleMore";
              toggleLink.setAttribute("class", "toggle-more-links");
              linkCell.appendChild(toggleLink);
            }
            linkCell.appendChild(link);
          });
        });
        document.querySelectorAll('a.toggle-more-links').forEach(function(toggleLink) {
          toggleLink.onclick = function() {
            var expanded = toggleLink.parentElement.getAttribute("data-more-links");
            toggleLink.parentElement.setAttribute("data-more-links", "on" === expanded ? "off" : "on");
            return false;
          };
        });
      };
    </script>
  </head>
  <body>
    <h1>{{- escapeXML ( index . 0 ).Target }} - Trivy Report - {{ now }}</h1>
    <table>
    {{- range . }}
      <tr class="group-header"><th colspan="6">{{ .Type | toString | escapeXML }}</th></tr>
      {{- if (eq (len .Vulnerabilities) 0) }}
      <tr><th colspan="6">No Vulnerabilities found</th></tr>
      {{- else }}
      <tr class="sub-header">
        <th>Package</th>
        <th>Vulnerability ID</th>
        <th>Severity</th>
        <th>Installed Version</th>
        <th>Fixed Version</th>
        <th>Links</th>
      </tr>
        {{- range .Vulnerabilities }}
      <tr class="severity-{{ escapeXML .Vulnerability.Severity }}">
        <td class="pkg-name">{{ escapeXML .PkgName }}</td>
        <td>{{ escapeXML .VulnerabilityID }}</td>
        <td class="severity">{{ escapeXML .Vulnerability.Severity }}</td>
        <td class="pkg-version">{{ escapeXML .InstalledVersion }}</td>
        <td>{{ escapeXML .FixedVersion }}</td>
        <td class="links" data-more-links="off">
          {{- range .Vulnerability.References }}
          <a href={{ escapeXML . | printf "%q" }}>{{ escapeXML . }}</a>
          {{- end }}
        </td>
      </tr>
        {{- end }}
      {{- end }}
      {{- if (eq (len .Misconfigurations ) 0) }}
      <tr><th colspan="6">No Misconfigurations found</th></tr>
      {{- else }}
      <tr class="sub-header">
        <th>Type</th>
        <th>Misconf ID</th>
        <th>Check</th>
        <th>Severity</th>
        <th>Message</th>
      </tr>
        {{- range .Misconfigurations }}
      <tr class="severity-{{ escapeXML .Severity }}">
        <td class="misconf-type">{{ escapeXML .Type }}</td>
        <td>{{ escapeXML .ID }}</td>
        <td class="misconf-check">{{ escapeXML .Title }}</td>
        <td class="severity">{{ escapeXML .Severity }}</td>
        <td class="link" data-more-links="off"  style="white-space:normal;">
          {{ escapeXML .Message }}
          <br>
            <a href={{ escapeXML .PrimaryURL | printf "%q" }}>{{ escapeXML .PrimaryURL }}</a>
          </br>
        </td>
      </tr>
        {{- end }}
      {{- end }}
    {{- end }}
    </table>
{{- else }}
  </head>
  <body>
    <h1>Trivy Returned Empty Report</h1>
{{- end }}
  </body>
</html>
```

2-**Generate Report Using the Custom Template**

Now, to generate an HTML report using the custom template, run the following command:
```
trivy fs --security-checks vuln,config,secret /home/aress/ -f template --template "@html.tpl" -o report.html
```
This command will use the `html.tpl` template you just created and output the report to `report.html`.
