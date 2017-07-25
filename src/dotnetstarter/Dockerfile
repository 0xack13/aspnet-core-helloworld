FROM microsoft/dotnet 

# Set ASPNETCORE_URLS
ENV ASPNETCORE_URLS=https://*:8080

# Switch to root for changing dir ownership/permissions
USER 0

# Copy the binaries
COPY /bin/release/netcoreapp1.0/publish app

# Change to app directory
WORKDIR app

# Set password for the certificate as 1234
# I'm using Environment Variable here to simplify the .NET Core sample.
ENV certPassword 1234

# Use opnssl to generate a self signed certificate cert.pfx with password $env:certPassword
RUN openssl genrsa -des3 -passout pass:${certPassword} -out server.key 2048
RUN openssl rsa -passin pass:${certPassword} -in server.key -out server.key
RUN openssl req -sha256 -new -key server.key -out server.csr -subj '/CN=localhost'
RUN openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt
RUN openssl pkcs12 -export -out cert.pfx -inkey server.key -in server.crt -certfile server.crt -passout pass:${certPassword}

# In order to drop the root user, we have to make some directories world
# writable as OpenShift default security model is to run the container under
# random UID.
RUN chown -R 1001:0 /app && chmod -R og+rwx /app

# Expose port 8080 for the application.
EXPOSE 8080

# Run container by default as user with id 1001 (default)
USER 1001

# Start the application using dotnet!!!
ENTRYPOINT dotnet dotnetstarter.dll
