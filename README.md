# Setup 
- Front end - React
- Back end - NVIDIA Triton Inference Server
- NVIDIA Triton Inference Server does not support CORS.
- NVIDIA Triton Inference Server URL - http://10.21.19.100:8000/v2/models/english2indic/versions/3/infer
- React is using AXIOS and running on http://10.21.19.200:7003.
- NGINX proxy is running on port number 9000.

## Install Nginx.

```sh
sudo apt update
sudo apt install nginx -y
```

## Configure Nginx as a Reverse Proxy.

```sh
sudo nano /etc/nginx/sites-available/triton-proxy
```

## Add following details.

```sh
server {
    listen 9000;

    location / {
        proxy_pass http://10.21.19.100:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # CORS Headers
        add_header 'Access-Control-Allow-Origin' 'http://10.21.19.200:7003';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
    }

    # Handle OPTIONS requests separately
    location / {
        if ($request_method = OPTIONS) {
            add_header 'Access-Control-Allow-Origin' 'http://10.21.19.200:7003';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
            return 204;
        }
    }
}
```

## Enable the Configuration.

```sh
sudo systemctl restart nginx
```

## Restart NGINX server.

```sh
sudo systemctl restart nginx
```

## Update React Axios requests.

```sh
axios.post("http://10.21.19.100:9000/v2/models/english2indic/versions/3/infer", data, {
    headers: {
        "Content-Type": "application/json"
    }
})
.then(response => {
    console.log(response.data);
})
.catch(error => {
    console.error("Error:", error);
});
```
