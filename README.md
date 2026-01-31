# DynamoDB Dashboard

A Web GUI Dashboard for local or remote [DynamoDB](https://aws.amazon.com/blogs/aws/dynamodb-local-for-desktop-development/) with features for querying, filtering, and data manipulation.

![npm](https://img.shields.io/npm/v/dynamodb-dashboard?label=npm&style=flat-square) 
![npm](https://img.shields.io/npm/dw/dynamodb-dashboard?style=flat-square) 
![npm](https://img.shields.io/npm/l/dynamodb-dashboard?style=flat-square) 
![Docker Pulls](https://img.shields.io/docker/pulls/kritishdhaubanjar/dynamodb-dashboard?style=flat-square) 
![Docker Image Size](https://img.shields.io/docker/image-size/kritishdhaubanjar/dynamodb-dashboard/latest?style=flat-square)

[![dynamodb-dashboard Builder](https://img.shields.io/github/actions/workflow/status/kritish-dhaubanjar/dynamodb-dashboard/build.yml?branch=main&label=dynamodb-dashboard%20Builder&style=flat-square)](https://github.com/kritish-dhaubanjar/dynamodb-dashboard/actions/workflows/build.yml)

---

## Features

- 📊 **Visual Interface** — Intuitive GUI for managing DynamoDB tables
- 🔍 **Query Builder** — Build and execute DynamoDB queries visually
- 🎯 **Data Filtering** — Filter and sort data with custom criteria
- 📄 **Bulk Operations** — Perform batch operations on multiple items
- 🔄 **Real-time Updates** — Support for DynamoDB Streams and SSE
- 🔐 **Secure Credentials** — AWS SDK v2 credential chain with multiple sources
- 🐳 **Docker Ready** — Containerized deployment with minimal setup

## Use Cases

**For Developers:** Quickly inspect and debug DynamoDB tables during development  
**For DBAs:** Visual data exploration without writing CLI commands  
**For QA:** Validate data integrity and test queries visually  
**For DevOps:** Manage remote DynamoDB instances from local dashboard

---

## Installation

### NPM (Recommended)
```bash
npm install --global dynamodb-dashboard
dynamodb-dashboard start
```

### Docker
```bash
docker pull kritishdhaubanjar/dynamodb-dashboard:latest
docker run -p 4567:4567 \
  -e AWS_REGION=us-west-2 \
  -e AWS_ENDPOINT=http://localhost:8000 \
  kritishdhaubanjar/dynamodb-dashboard:latest
```

### Clone & Build
```bash
git clone https://github.com/kritish-dhaubanjar/dynamodb-dashboard.git
cd dynamodb-dashboard
make watch
```

---

## Configuration

### Environment Variables
```bash
export AWS_REGION=us-west-2
export AWS_ENDPOINT=http://localhost:8000
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_SESSION_TOKEN=your-session-token  # optional
```

### AWS Credentials Resolution

The dashboard uses [AWS SDK for JavaScript (v2)](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials-node.html), which supports multiple credential sources (priority order):

1. ✅ **Explicit environment variables** (highest priority)
2. ✅ **~/.aws/credentials** file
3. ✅ **AWS SSO sessions** (via IAM identity center)
4. ✅ **EC2/ECS instance metadata** (for AWS-hosted deployments)
5. ✅ **Shared credentials file** (AWS_SHARED_CREDENTIALS_FILE)

---

## Development

### Setup Vue.js App
```bash
cd app
npm install
npm run dev
```

### Setup Node Express Server
```bash
cd server
npm install
npm start
```

### Watch Mode
```bash
cd dynamodb-dashboard
make watch
```

---

## Options

| Option | Description | Default |
|---------|-------------|---------|
| `-d, --debug` | Enable debug logging | `false` |
| `-p, --port` | Port to run app on | `4567` |
| `-h, --host` | Host to connect to | `127.0.0.1` |

---

## Deployment

### NGINX Configuration

For EventSource (Server-Sent Events) support, ensure NGINX handles long-lived HTTP connections:

```nginx
server {
    listen 80;
    server_name _;

    proxy_read_timeout 1d;
    proxy_send_timeout 1d;
    proxy_connect_timeout 1d;

    location / {
        proxy_pass http://localhost:4567;
    }

    location /dynamodb/api/database/stream/ {
        proxy_buffering off;
        proxy_cache off;
        chunked_transfer_encoding off;

        add_header Content-Type text/event-stream;
        add_header Cache-Control no-cache;
        add_header Connection keep-alive;

        proxy_pass http://localhost:4567;
    }
}
```

---

## Screenshots

### Main Dashboard
![Main dashboard view](https://user-images.githubusercontent.com/25634165/213922274-d70cde00-4d70-47ac-ab84-68b6f0933d58.png)

### Query Builder
![Query builder interface](https://user-images.githubusercontent.com/25634165/215118400-fe18ea87-5562-4e7d-be7b-ccf61a3fbe99.png)

### Data Filtering
![Filter and sort options](https://user-images.githubusercontent.com/25634165/192109458-a621bc06-788d-4d54-9dc2-9064380ee837.mp4)

---

## Demo

**Interactive Preview:** https://user-images.githubusercontent.com/25634165/192109458-a621bc06-788d-4d54-9dc2-9064380ee837.mp4

---

## Troubleshooting

### Connection Issues
- **"Unable to connect"** — Verify `AWS_ENDPOINT` and `AWS_REGION` match your DynamoDB setup
- **"Credentials not loading"** — Check that environment variables are exported before starting
- **"Port already in use"** — Use `-p` flag to specify a different port

### NGINX Timeouts
If experiencing SSE disconnections, increase `proxy_read_timeout` and `proxy_send_timeout` to `2d` (2 days).

---

## Contributing

Contributions welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

Distributed under the MIT License. See [LICENSE](LICENSE) for details.
