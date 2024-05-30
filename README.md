# Ansible Role: Traefik

An Ansible Role that installs [Traefik](https://traefik.io/traefik/) in Docker.

## Requirements

Docker Daemon configured in target hosts

## Role Variables

Role variables can be divided into three groups depending on their purpose - for container, for traefik.yml. for providers.
Available variables are listed below, along with default values (see `defaults/main.yml`)

Traefik container configuration variables. Docker-compose configuration is compiled based on these variables, so some of them look like docker-compose.yml options. 

    ### Traefik docker container configs
    traefik_image: traefik:v2.2
    traefik_container_name: traefik

Traefik container can access all of host interfaces and host ports. It can be usefull when you will use traefik to route internet trafic in not dockerized services, who listen only hosts 127.0.0.1 interface. 

    # That option allow you to setup traefik with "network_mode: host"
    traefik_host_network_mode_host: False

Following variables are completely identical to the docker-compose.yml settings

    # Docker compose like yml dict of networks
    # Traefik container will attach to all of them
    # Example:
    # traefik_container_networks:
    #   default:
    #     external: True
    #   frontend:
    #     driver: custom-driver-1
    #   backend:
    #     driver: custom-driver-2
    #     driver_opts:
    #       foo: "1"
    #       bar: "2"
    traefik_container_networks: {}

    # Docker compose like yml list of container labels
    # Thats labels only for traefik container
    # Example:
    # traefik_container_labels:
    #   - "traefik.enable=true"
    #   - "traefik.http.routers.traefik.entrypoints=https"
    #   - "traefik.http.routers.traefik.rule=Host(`traefik.example.com`)"
    #   - "traefik.http.routers.traefik.tls=true"
    #   - "traefik.http.routers.traefik.tls.certresolver=letsEncrypt"
    traefik_container_labels: []

    # Docker compose like yml list of command options for traefik
    # Example:
    # traefic_commands:
    # - "--api.insecure=true"
    # - "--providers.docker"
    # - "--providers.docker.exposedByDefault=false"
    # - "--providers.docker.network=proxynet"
    # - "--entrypoints.http.address=:80"
    # - "--entrypoints.http.http.redirections.entrypoint.to=https"
    # - "--entrypoints.http.http.redirections.entrypoint.scheme=https"
    # - "--entrypoints.https.address=:443"
    # - "--log.level=DEBUG"
    traefic_commands: []

traefik.yml configuration variables. These variables are used to build the main configuration file known as `traefik.yml`.

    # Path to traefik directory on host
    traefik_dir: /var/lib/traefik

    # Enable traefik dashboard?
    traefik_dashboard: False

The variables presented below contain completely identical options from the traefik.yml. To configure them, it is recommended to read the official documentation in the links provided with the variables.

    # Dict with entryPoints configuration
    # https://doc.traefik.io/traefik/routing/entrypoints/
    traefik_entrypoints:
      http:
        address: ":80"
      https:
        address: ":443"

    # Dict with routers
    # https://doc.traefik.io/traefik/routing/routers/
    traefik_routers:   
      http-catchall:
        rule: hostregexp(`{host:.+}`)
        entrypoints:
          - http
        middlewares:
          - redirect-to-https

    # Dict with middlewares
    # https://doc.traefik.io/traefik/middlewares/overview/
    traefik_middlewares:   
      redirect-to-https:
        redirectScheme:
          scheme: https
          permanent: false

    # Should TLS be configured?
    traefik_tls: True
    traefik_tls_acme_mail: nobody@mail.ru
    traefik_tls_acme_storage: "{{ traefik_dir }}/acme.json"
    traefik_tls_challenge:
      # TLS challenges can be httpChallenge or tlsChallenge.
      # dnsChallenge currently unsupported
      type: tlsChallenge
      # entryPoint MUST be provided with httpChallenge
      # entryPoint: None


Variables for providers. Currently only file and docker providers are available.

Docker provider configuration.

    # Provider configuration in main traefik.yml configuration file
    # https://doc.traefik.io/traefik/providers/docker/
    traefik_docker_provider: 
      endpoint: "unix:///var/run/docker.sock"

File provider confs

    # Provider configuration in main traefik.yml configuration file
    # Currently supported only `filename` and `watch` options for provider
    # https://doc.traefik.io/traefik/providers/file/
    traefik_file_provider: {}
    # Routers for file proider
    traefik_fp_routers: {}
    # Middlewares for file proider
    traefik_fp_middlewares: {}
    # Services for file proider
    traefik_fp_services: {}


## Example Playbook

```yaml
- hosts: all
  roles:
    - one_mind.traefik
```

## License

MIT / BSD

## Author Information

This role was created in 2024 by Dmitry Razin
