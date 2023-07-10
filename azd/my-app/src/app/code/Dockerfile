# Use an official .NET Core SDK image as a build environment
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build-env

# Set the working directory to /app
WORKDIR /app

# Copy the csproj file and restore any dependencies
COPY *.csproj ./
RUN dotnet restore

# Copy the rest of the app's source code and build the app
COPY . ./
RUN dotnet publish -c Release -o out

# Use an official .NET Core runtime image as the base image
FROM mcr.microsoft.com/dotnet/aspnet:6.0

# Set the working directory to /app
WORKDIR /app

# Copy the app's build output from the build environment to the container
COPY --from=build-env /app/out .

# Expose the port that the app listens on
EXPOSE 80

# Set the entry point for the container to the app's executable
ENTRYPOINT ["dotnet", "WebApp-OpenIDConnect-DotNet.dll"]