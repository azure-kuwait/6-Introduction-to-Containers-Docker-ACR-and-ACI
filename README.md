# 6-Introduction-to-Containers-Docker-ACR-and-ACI
This repository contains the PowerPoint presentation and the code samples used in the meetup.

The source code in the repository is for the second demo of the session. It's based on Microsoft official learning module ["Build a containerized web application with Docker"](https://docs.microsoft.com/en-us/learn/modules/intro-to-containers/). You can access the original code [here](https://github.com/MicrosoftDocs/mslearn-hotel-reservation-system). We upgraded the original code to .NET Core 3.1 and presented a more clean Docker image using the "multi-stage builds" technique in the Dockerfile.

## How to execute Demo 2
---
1.	Use the source code found [src/mslearn-hotel-reservation-system](src/mslearn-hotel-reservation-system)

2.	Move to the src folder. (*below example assumes you are going to clone the repository to a folder has the repository name '6-Introduction-to-Containers-Docker-ACR-and-AC'*)
```bash
cd 6-Introduction-to-Containers-Docker-ACR-and-ACI/src/mslearn-hotel-reservation-system/src
```

3.	In this directory, create a new file named Dockerfile with no file extension and open it in a text editor like VS Code. On Windows, you can run the following commands:
```bash
cd . > Dockerfile
code .
```

4.	Add following to the Dockerfile

```Dockerfile
# Build image
FROM mcr.microsoft.com/dotnet/core/sdk:3.1-buster AS build-env
WORKDIR /src
COPY ["HotelReservationSystem/HotelReservationSystem.csproj", "HotelReservationSystem/"]
COPY ["HotelReservationSystemTypes/HotelReservationSystemTypes.csproj", "HotelReservationSystemTypes/"]
RUN dotnet restore "HotelReservationSystem/HotelReservationSystem.csproj"
COPY . .
WORKDIR "/src/HotelReservationSystem"
RUN dotnet build "HotelReservationSystem.csproj" -c Release -o /app
RUN dotnet publish "HotelReservationSystem.csproj" -c Release -o /app

# Runtime image
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-buster-slim
WORKDIR /app
EXPOSE 80
COPY --from=build-env /app .
ENTRYPOINT ["dotnet", "HotelReservationSystem.dll"]
```

5.	Build the Dockerfile to create an image with name "reservationsystem" \
`docker image build -t reservationsystem .`

6. Double check the image built successfully by listing the the images list. You have to see the new image called **reservationsystem** \
`docker image ls`

7.	Test the image by running a container \
`docker container run -p 8080:80 -d --name reservations reservationsystem`

8. Open any browser and go to \
[http://localhost:8080/api/reservations/1](http://localhost:8080/api/reservations/1)

9.	Show running containers \
`docker container ls`

10.	Stop the reservations container with the following command \
`docker container stop reservations`

11.	Delete the reservations container from the local registry \
`docker container rm reservations`

## How to execute Demo 3
---
1.	Create ACR from Portal or Azure CLI (*make sure replace all place holders \<resource-group-name> and \<registry-name>*)\
`az group create --name <resource-group-name> --location <resource-group-location>` \
`az acr create --name <registry-name> --sku Standard --resource-group <resource-group-name> --admin-enabled true`

2.	In your local command line, run the following command to tag the reservationsystem image with the name of your registry. Replace **\<registry-name>** with the name of your registry in Azure Container Registry \
`docker tag reservationsystem:latest <registry-name>.azurecr.io/reservationsystem:latest`

3.	Run the docker image ls command to verify that the image has been tagged correctly \
`docker image ls`

4.	Sign in to your registry in Azure Container Registry. Use the docker login command and specify the login server for the registry that you noted earlier. Enter the username and password for the registry when prompted \
`docker login <login-server>`

5.	Upload the image to the registry in Azure Container Registry with the docker push command \
`docker image push <registry-name>.azurecr.io/reservationsystem:latest`

6. Get registry's credentials and save the output to use it in the next step \
`az acr credential show --name <registry-name>`

7. Create Azure container instance (ACI) using following command (*make sure replace palceholders like \<aci-name>*) \
`az container create --name <aci-name> --resource-group <resource-group-name> --registry-login-server <registry-name>.azurecr.io --registry-username <registry-username> --registry-password <registry-password> --image  <registry-name>.azurecr.io/reservationsystem:latest --location <aci-location> --dns-name-label <unique-dns-name-label> --restart-policy Always`
 
 8. Show container's unique DNS name label \
 `az container show --name <aci-name> --resource-group <resource-group-name> --query "ipAddress.fqdn" --output tsv` \

    The output will be something like: \
    `azureq8hotelsysteminstance.westeurope.azurecontainer.io`


 9. Take the output and add it to `/api/reservations/1` the result will be something like: `azureq8hotelsysteminstance.westeurope.azurecontainer.io/api/reservations/1` copy this output

 10. Open any browser and paste the link. You should be able to see an output.