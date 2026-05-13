# CrushFTP for Kubernetes

Share your files securely with FTP, Implicit FTPS, SFTP, HTTP, or HTTPS using CrushFTP using Helm and docker.

See the full documentation at https://kakoluz.github.io/helm-crushftp/.

# Getting started with helm

Add the helm repository and install the chart.

```
helm repo add crushftp https://kakoluz.github.io/helm-crushftp
helm repo update

helm install crushftp crushftp/crushftp
```

## Helm chart values

Override helm chart values with the settings you want.

| Parameter                    | Description                                                                                                | Default      |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------- | ------------ |
| admin.username               | Username for the initial admin account.                                                                    | crushadmin   |
| admin.password               | Password for the initial admin account.                                                                    | *generated*  |
| admin.protocol               | Protocol for health checks and probs.                                                                      | http         |
| admin.port                   | Port for health checks and probs.                                                                          | 8080         |
| features.enableFtp           | Used to enable FTP protocol if needed.                                                                     | false        |
| tls.secretName               | Name of the secret to use for the TLS certificate.                                                         | crushftp-tls |
| volumes                      | Set of volumes from other sites or containers to mount.<br> Requires `name`, `claimName`, and `mountPath`. | [ ]          |
| configVolume.size            | Size of the CrushFTP configuration volume.                                                                 | 8Gi          |
| loadBalancerIp               | IP address of the ingress to use.                                                                          | 127.0.0.1    |
| shared.hosts.crushFtp.root   | Root domain of the sftp site.                                                                              | .local.com   |
| shared.hosts.crushFtp.prefix | Prefix or sub-domain of the sftp site.                                                                     | ftp          |
| shared.ingress.clusterIssuer | Used to enable a cluster certificate issuer such as cert-manager and lets-encrypt.                         | ''           |
| shared.storageClassName      | Optional storage class for the config volume. Leave empty to use cluster default.                          | ""         |
| image.repository             | Container image repository used by the chart.                                                              | ghcr.io/kakoluz/crushftp |
| image.tag                    | Container image tag. Defaults to chart appVersion.                                                         | "11"        |

# Docker Image

Docker image instructions if used separately from helm chart.

## Volumes

| Volume                | Required | Function                                       | Example                                  |
| --------------------- | -------- | ---------------------------------------------- | ---------------------------------------- |
| `/var/opt/CrushFTP11` | Yes      | Persistent storage for CrushFTP config         | `/your/config/path/:/var/opt/CrushFTP11` |
| `/mnt/FTP/Shared`     | No       | Shared host folder for file sharing with users | `/your/host/path/:/mnt/FTP/Shared`       |

* You can add as many volumes as you want between host and the container and change their mount location within the container. You will configure individual folder access and permissions for each user in CrushFTPs User Manager. The "/mnt/FTP/Shared" in the table above is just one such example.

## Ports

| Port        | Proto | Required | Function          | Example               |
| ----------- | ----- | -------- | ----------------- | --------------------- |
| `21`        | TCP   | Yes      | FTP Port          | `21:21`               |
| `443`       | TCP   | Yes      | HTTPS Port        | `443:443`             |
| `2000-2100` | TCP   | Yes      | Passive FTP Ports | `2000-2100:2000-2100` |
| `2222`      | TCP   | Yes      | SFTP Port         | `2222:2222`           |
| `8080`      | TCP   | Yes      | HTTP Port         | `8080:8080`           |
| `9090`      | TCP   | Yes      | HTTP Alt Port     | `9090:9090`           |

* If you wish to run certain protocols on different ports you will need to change these to match the CrushFTP config. If you enable implicit or explicit FTPS those ports will also need to be opened.

## Environment Variables

| Variable               | Description               | Default      |
| :--------------------- | :------------------------ | :----------- |
| `CRUSH_ADMIN_USER`     | Admin user of CrushFTP    | `crushadmin` |
| `CRUSH_ADMIN_PASSWORD` | Password for admin user   | `crushadmin` |
| `CRUSH_ADMIN_PROTOCOL` | Protocol for health cecks | `http`       |
| `CRUSH_ADMIN_PORT`     | Port for health cecks     | `8080`       |

## Installation

Run this container and mount the containers `/var/opt/CrushFTP11` volume to the host to keep CrushFTP's configuration persistent. Open a browser and go to `http://<IP>:8080`. Note that the default username and password are both `crushadmin` unless the default environment variables are changed.

This command will create a new container and expose all ports. Remember to change the `<volume>` to a location on your host machine.

```
docker run -p 21:21 -p 443:443 -p 2000-2100:2000-2100 -p 2222:2222 -p 8080:8080 -p 9090:9090 -v <volume>:/var/opt/CrushFTP11 ghcr.io/kakoluz/crushftp:11
```

# CrushFTP Configuration

Visit the [CrushFTP 11 Wiki](https://www.crushftp.com/crush11wiki/)


# Contributing

1. Install Docker from https://www.docker.com/get-started
2. Build and start the image by running the following command:

    ```bash
    docker-compose up --build
    ```

## Publishing docker image

Docker image publishing is automated with GitHub Actions.

1. Push to `main` to publish `ghcr.io/<owner>/crushftp:11`, `:latest`, and `:sha-<commit>` tags.
2. Push a release tag like `v1.1.0` to additionally publish the git tag image.
3. Pull requests build and validate the image without pushing.

## Publishing helm chart

Helm chart publishing is automated with GitHub Actions.

1. Update `charts/crushftp/Chart.yaml` with the target chart version.
2. Create and push a matching git tag in the format `v<chart-version>`.
3. The `Release Helm Chart` workflow packages the chart, uploads it to the release, and updates `docs/index.yaml`.

# References

- Docker image based on https://github.com/MarkusMcNugen/docker-CrushFTP
- [CrushFTP - Linux Install](https://www.crushftp.com/crush11wiki/Wiki.jsp?page=Linux%20Install)
