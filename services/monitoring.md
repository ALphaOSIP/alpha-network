# Monitoring Services

## Uptime Kuma

- **Type:** HTTP monitoring dashboard
- **URL:** [http://10.0.1.100:3001](http://10.0.1.100:3001)
- **Purpose:** Monitors all network services
- **Notifications:** Push alerts via Ntfy

## Ntfy

- **Type:** Push notification server
- **URL:** [http://10.0.1.100:2586](http://10.0.1.100:2586)
- **Purpose:** Sends alerts to phone

## Custom Health Check Scripts

- **Type:** Shell scripts
- **Purpose:** Check all services and send alerts automatically

## Portainer

- **Type:** Docker management UI
- **Purpose:** Container management dashboard
- **HTTP:** [http://10.0.1.100:9000](http://10.0.1.100:9000)
- **HTTPS:** [https://10.0.1.100:9443](https://10.0.1.100:9443)
