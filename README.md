# PERN Microservices

A simplified microservices template for use with docker-compose.
Each service is separated by a network, ensuring local services
only have access to their local database within their own network.

Services are expected to communicate over their APIs, maintaining
a copy of their own database entries if necessary. This means
each service built within the architecture is modular, it can be
removed or replaced and still allow the other services to function.

An Nginx gateway routes to different services on different ports
using the Nginx configuration. Not only is this method of direct
routing faster, it also acts as protection from the greater web.

## Usage

To get started, make sure you have [Docker installed](https://docs.docker.com/docker-for-mac/install/) on your system, and then clone this repository.

Containers can be brought up with the docker-compose.dev.yml by running the following command: <code>docker-compose -f docker-compose.dev.yml up --build -d</code>

only exposed endpoints are available from your localhost, using: <code>docker ps</code> you can see which ports are available from your localhost
by the 0.0.0.0 address. Other ports are only available within the docker networks, these can communicate via their container names over the docker
network, but it is not necessary to expose them to the localhost.

Data is persisted to the database via volumes. You can add more services by adding them to the docker-compose file, along wth updating
the nginx.conf file to include the routes to expose. By default this is set up for a single service, you may wish to set up routes
for each service named as ````/service````. This is performed
in the nginx conf by using the container name followed by the port:

<pre>
<code>
    location /api/ {
        proxy_pass http://artemis-api:4000/;
    }
</code>
</pre>

To add more services you may wish to change the config to something more like:

<pre>
<code>
server {
    listen 80;

    server_name localhost;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Connection $http_connection;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Sec-WebSocket-Extensions $http_sec_websocket_extensions;
        proxy_set_header Sec-WebSocket-Key $http_sec_websocket_key;
        proxy_set_header Sec-WebSocket-Version $http_sec_websocket_version;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 900;
        client_max_body_size 0;
        proxy_buffering off;
        add_header X-Accel-Buffering no;

        proxy_pass http://gateway-service:3000/;
    }

    location gateway/api/ {
        proxy_pass http://gateway-service-api:4000/;
    }
}
</code>
</pre>

## Package Manager

The package manager used by default is ````yarn````. Yarn is somewhat faster than ```npm``` when initially installing a large number of packages. We also prefer the syntax since you no longer need the ````run```` keyword like you do when using ````npm````.

## Exposed Ports

- **nginx** - `:8080`
- **adminer** - `:8081`

## Commands

- ````docker-compose -f docker-compose.dev.yml up --build -d ````

## Technologies Included

- React
- Node
- TypeScript
- Webpack

## Git Setup

We recommend using different repositories for each service, and
then having separate repositories for the API and Client. Whilst each
service is packaged together with the API and Client, it is important
to recognise that these themselves are separate modular sub-services. The API
should not be dependent on the client of vice-versa. You can achieve this by initialising each as it's own repo, then reinisitalising the main repo.

For a basic application we recommend keeping the Client and Server as separate services. This is how the repository is configured as default. For a more advanced setup (when multiple services are invovled) I would recommend creating a directory for each service in the <code>services</code> folder, you should then create your <code>client</code> and <code>api</code> for each of these services within their own project folders, changing the paths within the <code>docker-compose.yml</code> as required. To add more services you can easily duplicate the <code>client</code>
and <code>api</code> services, I recommend naming each service as <code>{service-name}-client}</code> and <code>{service-name}-api</code> respectively.

When adding services you will also need to update the nginx.config to route the request on a per-service basis. We use Nginx as our gateway for requests, this is set up as reverse proxy to keep our internal requests secure.

## Child Repositories

These repositores are the subrepositories used in this tempalte. These are setup to work out of the box
with the PERN stack. You can use these as templates for your own services, or create our own from scratch.

- [Node API](https://github.com/PlanetDebug/PERN-API)
- [React Client](https://github.com/PlanetDebug/PERN-CLIENT)
