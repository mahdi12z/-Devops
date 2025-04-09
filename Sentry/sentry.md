### **Installing Sentry Self-Hosted Using GitHub Release**

#### **1. Download and Install the Latest Sentry Version**

1. **Download the Latest Version:** First, use `curl` to fetch the latest version of Sentry:
```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/getsentry/self-hosted/releases/latest)
VERSION=${VERSION##*/}
```

 ###  2.**Clone the Git Repository:** Clone the `self-hosted` repository from GitHub:
```bash
git clone https://github.com/getsentry/self-hosted.git
cd self-hosted
```

 ###  3.**Checkout the Latest Version:** Then, checkout the version obtained from `VERSION`:
```bash
git checkout ${VERSION}
```

 ### 4. **Run the Installation Script:** To install Sentry, run the following script:
```bash
./install.sh
```
#### **2. Start Sentry with Docker Compose**

After a successful installation, use the following command to start Sentry:
```bash
`docker compose up --wait`
```

#### **3. Edit Configuration Files**

1. **Edit `config.yml` File:** For configuring Sentry, edit the `config.yml` file. In this file, settings related to URL and other configurations are applied.
    
    Specifically, we set values like `system.url-prefix` and `system.internal-url-prefix`.
    
    **Example:**
 ```bash
system.url-prefix:  'http://192.168.100.240:9000'
system.internal-url-prefix: 'http://web:9000'
```

2.**Edit `.env` File:** Edit the `.env` file to set environment variables required for Docker and other configurations. In this step, we made specific changes like setting `SENTRY_BIND` for the server address and port.

**Example:**

```bash
SENTRY_BIND=192.168.100.240:9000
```

#### **4. Restart and Check Container Status**

1. **Check Container Status:** To check the status of running containers, use the following command:
 ```
 `docker ps`
```
  

2. **Restart Containers:** If a restart is required, use the following commands:

```bash
`docker compose down
docker compose up --wait`
```
