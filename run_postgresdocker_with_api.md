Create project
Add NuGet packages
Create Models and DbContext
No connection string in `appsettings.json`
  - the connection string is left empty because the docker-compose will propagate the connection string
  - since we will use a "containerized database"
Run migrations: `dotnet ef add migrations "initial-migrations"`
Then check with: `dotnet build` (inside the .csproj directory)
Create Services and Controllers with Endpoints

Create Dockerfile (along with the solution file, sln-level directory)
  - create `.dockerignore` file also
Build the docker image: `docker build . -t demoapp`
Then run it: `docker run -p 8081:80 -e ASPNETCORE_URLS=http://+:80 demoapp

THEN create a `docker-compose.yml` file
```yml
version: "3.9"

networks:
  dev:
    driver: bridge

services:
  demoapp:
    image: demoapp
    depends_on: 
      - "app_db"
    container_name: demoapp-services
    ports:
      - "8088:80"
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      - ConnectionStrings__DefaultConnection=User ID=postgres;Password=postgres;Server=app_db;Port=5432;Database=SampleDbDriver; IntegratedSecurity=true;Pooling=true
      - ASPNETCORE_URLS=http://+:80
    networks: 
      - dev
  
  app_db:
      image: postgres:latest
      container_name: app_db
      environment:
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
        - POSTGRES_DB=SampleDbDriver
      ports:
        - "5433:5432"
      restart: always
      volumes:
        - app_data:/var/lib/postgresql/data
      networks: 
        - dev

volumes: 
  app_data:
```
NOTICE:
  - get the image name for the API from `docker build . -t <container_name>
  - the volumes is "app_data"
  - networks dev is bridge (search this)
  - the ports mapped are different

THEN run the docker-compose file: `docker-compose up`
- now you have the database running and api/application running
Connect to the Database Connection using DBeaver
  - Port is 5433
  - user and pw: postgres
  - database: postgres (for now)

DO THIS WHILE THE CONTAINER IS RUNNING:
NOW get the Connection String in docker-compose file
AND paste it to the `appsettings.json`
  - modify/change the ff to:
    - Server=localhost
    - Port=5433
  - this is done so we can access the containerized database in our localhost
Create a new terminal
  - Then run: `dotnet build`
  - Then run database command script manually: `dotnet ef database update`
    - ^ has an automatic database command script that updates databases, tags, releases, containers, etc. to latest changes(research about this)

Go to DBeaver to see the database
  - right-click postgres or create a new connection
  - Edit Connection:
    - change database name to "SampleDbDriver"
    - Test connection and Ok
  - Navigate and open <Db Name> -> Schemas -> public -> tables -> <can see all tables here>

Then just delete or remove the connection string (leave it as empty string "")
  - in `appsettings.json`
  - or just leave it (not remove it)

Stop the container and run `docker-compose down` to completely remove it
Rebuild the whole container with: `docker-compose up --build`

Use Postman or any Http Client
  - connect to localhost:8088/drivers
  - send a request and see that it is now connected with 200 OK response
See the database changes in DBeaver by refreshing the table

Now it all works as expected

`
