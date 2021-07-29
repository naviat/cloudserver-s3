You can set credentials for many accounts by editing conf/authdata.json (see below for further info), but if you just want to specify one set of your own, you can use these environment variables.

The default access key is `newAccessKey`, with the secret key `newSecretKey`

```yaml
  environment:  
    - SCALITY_ACCESS_KEY_ID=newAccessKey
    - SCALITY_SECRET_ACCESS_KEY=newSecretKey  
```

**In production with Docker hosted CloudServer**

*In production, we expect that data will be persistent, that you will use the multiple backends capabilities of Zenko CloudServer, and that you will have a custom endpoint for your local storage, and custom credentials for your local storage:*

```yaml
  volumes:  
    - s3data:/usr/src/app/localData  
    - s3metadata:/usr/src/app/localMetadata  
volumes:  
  s3data:
  s3metadata:
```

*For launch the s3 server in this context, copy the YAML file docker-compose.yml in a directory of your choice*  
`docker-compose up -d`

*For stop the s3server and remove the container Docker*  

`docker-compose down`

*For container logs*  

`docker-compose logs -f`

*For logs nodejs in the container*  
 
`docker-compose logs | head -n +25`
