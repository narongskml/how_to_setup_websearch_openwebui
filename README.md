How to install SearXNG   for  using with Open WebUI


1. create folder for project  example  d:\project\webui_search
2. create sub folder  searxng     d:\project\webui_search\searxng
3. create file .env

   ```
   # SearXNG
   SEARXNG_HOSTNAME=localhost:8080/
   ```
4. cd to folder searxgn

   1. create file settings.yml

      ```yaml
      # see https://docs.searxng.org/admin/settings/settings.html#settings-use-default-settings
      use_default_settings: true

      server:
        # base_url is defined in the SEARXNG_BASE_URL environment variable, see .env and docker-compose.yml
        secret_key: "ul01f2235e4db22c15d61df5f994405f180cbd8f40de31b7a1141e3efb8b0f07f7trasecretkey"  # change this!
        limiter: true  # can be disabled for a private instance
        image_proxy: true
        port: 8080
        bind_address: "0.0.0.0"

      ui:
        static_use_hash: true

      search:
        safe_search: 0
        autocomplete: ""
        default_lang: ""
        formats:
          - html
          - json # json is required
        # remove format to deny access, use lower case.
        # formats: [html, csv, json, rss]
      redis:
        # URL to connect redis database. Is overwritten by ${SEARXNG_REDIS_URL}.
        # https://docs.searxng.org/admin/settings/settings_redis.html#settings-redis
        url: redis://redis:6379/0
      ```
   2. create file limiter.toml

      ```toml
      # This configuration file updates the default configuration file
      # See https://github.com/searxng/searxng/blob/master/searx/botdetection/limiter.toml

      [botdetection.ip_limit]
      # activate link_token method in the ip_limit method
      link_token = false

      [botdetection.ip_lists]
      block_ip = []
      pass_ip = []
      ```
   3. create file uwsig.ini
   4. ```ini
      [uwsgi]
      # Who will run the code
      uid = searxng
      gid = searxng

      # Number of workers (usually CPU count)
      # default value: %k (= number of CPU core, see Dockerfile)
      workers = 1

      # Number of threads per worker
      # default value: 4 (see Dockerfile)
      threads = 4

      # The right granted on the created socket
      chmod-socket = 666

      # Plugin to use and interpreter config
      single-interpreter = true
      master = true
      plugin = python3
      lazy-apps = true
      enable-threads = 4

      # Module to import
      module = searx.webapp

      # Virtualenv and python path
      pythonpath = /usr/local/searxng/
      chdir = /usr/local/searxng/searx/

      # automatically set processes name to something meaningful
      auto-procname = true

      # Disable request logging for privacy
      disable-logging = true
      log-5xx = true

      # Set the max size of a request (request-body excluded)
      buffer-size = 8192

      # No keep alive
      # See https://github.com/searx/searx-docker/issues/24
      add-header = Connection: close

      # uwsgi serves the static files
      static-map = /static=/usr/local/searxng/searx/static
      # expires set to one day
      static-expires = /* 86400
      static-gzip-all = True
      offload-threads = 4
      ```
5. Create file docker-compose.yml

   ```yaml
   services:
     redis:
       container_name: redis
       image: docker.io/valkey/valkey:8-alpine
       command: valkey-server --save 30 1 --loglevel warning
       restart: unless-stopped
       networks:
         - searxng
       volumes:
         - valkey-data:/data
       cap_drop:
         - ALL
       cap_add:
         - SETGID
         - SETUID
         - DAC_OVERRIDE
       logging:
         driver: "json-file"
         options:
           max-size: "1m"
           max-file: "1"

     searxng:
       container_name: searxng
       image: docker.io/searxng/searxng:latest
       restart: unless-stopped
       networks:
         - searxng
       ports:
         - "8080:8080" #change 8181 as needed, but not 8080
       volumes:
         - ./searxng:/etc/searxng:rw
       environment:
         - SEARXNG_BASE_URL=http://host.docker.internal:8080/ #Change "your.docker.server.ip" to your Docker server's IP
         - UWSGI_WORKERS=4 #You can change this
         - UWSGI_THREADS=4 #You can change this
       cap_drop:
         - ALL
       cap_add:
         - CHOWN
         - SETGID
         - SETUID
       logging:
         driver: "json-file"
         options:
           max-size: "1m"
           max-file: "1"

   networks:
     searxng:

   volumes:
     valkey-data: #redis storage
     searxng: #searxng storage
   ```
6. run command

   ```
   docker compose up -d
   ```
