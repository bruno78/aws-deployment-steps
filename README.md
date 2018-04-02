# AWS SITE DEPLOYMENT 

1. click on the services
2. Click on the Storage S3
3. Create a bucket 
4. Name a bucket and hit next skipping all the steps until you create the bucket
5. Click on the bucket and then choose properties tab 
6. Choose Static website hosting and choose "Use this bucket to host a website"

## REACT SIDE

1. Create a .env file in the root folder and add this local variable
``` 
    REACT_APP_HOST=http://localhost:8080
```
Note: YOU GOTTA HAVE *_REACT_APP_*

2. Remove proxy containng http://localhost:8080 from package.json
3. Remove `registerServiceWorker();` from index.js file 
4. Fixing axios in the app.js

```
await axios.get(process.env.REACT_APP_HOST +'/users')
```
NOTE: do this for all the requests involving the link

## BACK to AMAZON

5. Config AWS CLI. 
6. Go to AWS console at the website and choose the IAM console
7. Hit next permissions and Create a user group 
8. Name the group: 'SecaAdmins'
9. Once it's createad click on the SecaAdmin
10. Choose users
11. Choose new user 
12. Add a user name and choose Progammatic access hit next
13. you should have the access key ID and Secret key access 
14. Save the key id and secret key access 
15. Close it

## COMMAND LINE 

16. At the command line type `aws configure`
17. Add your keys and region:
``` 
    AWS Access Key ID [None]: **********************BQA
    AWS Secret Access Key [None]: **********************3IV
    Default region name [None]: us-east-1
    Default output format [None]: json 
```
Note: 
18. npm install 
19. npm run build
20. Sanity check, type on command line: `aws s3 ls` It should return a list of buckets available
21. Sync the folder with the bucket: `aws s3 sync build/ s3://bucket-seca-bruno`

NOTE IF YOU GOT ACCESS ERROR:

Go to your bucket and choose overview and select upload: 
Choose upload and upload all the content from inside of your build directory

Once is done, choose properties and click on Static website hosting and you might see the link there, click on it and you will see your website hosting. 


## Updating and configure your server for scalability 

1. Go to EC2 and choose launch instance 
2. Choose Ubuntu Server 16.04
3. Choosing the free tier for Microservices is not good... So choose `T2 Large`
4. Don't choose nothing on step 3
5. Step 4 choose 16 Gb
6. Step 5 add tags don't add one now
7. Step 6 Configuring SSH 
    - Add a rule 'Custom TCP' 8080 (for our API gateway)
    - Add a rule 'Custom TCP' 8761 (for Eureka)
    - Add a rule 'Custom TCP' 5432 (for the database)

NOTE: Make sure every source field is set to:
0.0.0.0/0, ::/0

8. Review and launch
9. It's going to as for a key pair. Choose create a key pair and add `key-pair` and download it.
10. Launch and see. 
11. At the bottom of the screen you will PUBLIC DNS (IPV4) copy that and:

## Back to command line 

Once you've had your key-pair downloaded:
1. Type ssh -i ~/Downloads/seca-keypair.pem ubuntu@ec2-35-173-178-66.compute-1.amazonaws.com

Don't forget to add ubuntu@ to your public DNS

if you run into permission problems, try:
chmod 400 ~/Downloads/seca-keypair.pem

and then run the command again
ssh -i ~/Downloads/seca-keypair.pem ubuntu@ec2-35-173-178-66.compute-1.amazonaws.com

You should see something like this in your terminal:

```ubuntu@ip-172-31-50-35```


## Downloading and Setting up Docker (ubuntu terminal)

1. Download docker:

```curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -```

```sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"```

```sudo apt-get update```

```sudo apt-get install -y docker-ce```

2. Check what version of Ubuntu you have: `lsb_release -a`

3. Installing docker-compose

```sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose```

```sudo chmod +x /usr/local/bin/docker-compose```

```docker-compose --version```

## Setting up the Backend (regular terminal)

1. Enable CORS:
    - Go to api-gateway folder of your backend app
    - type: `idea build.gradle
    - in the src folder in the main file: ZuulGatewayApi.java and add this:
    ```
    @Bean
    public CorsFilter corsFilter() {

        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        final CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        config.setAllowedOrigins(Collections.singletonList("*"));
        config.setAllowedHeaders(Collections.singletonList("*"));
        config.setAllowedMethods(Arrays.stream(Http.HttpMethod.values()).map(Http.HttpMethod::name).collect(Collectors.toList()));
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    ```

    You will endup with with this file:

    ```
    package com.example.apigateway;

    import com.netflix.ribbon.proxy.annotation.Http;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
    import org.springframework.context.annotation.Bean;
    import org.springframework.web.cors.CorsConfiguration;
    import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
    import org.springframework.web.filter.CorsFilter;

    import java.util.Arrays;
    import java.util.Collections;
    import java.util.stream.Collectors;

    @SpringBootApplication
    @EnableZuulProxy
    public class ZuulGatewayApplication {

        public static void main(String[] args) {
            SpringApplication.run(ZuulGatewayApplication.class, args);
        }

        @Bean
        public CorsFilter corsFilter() {

            final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
            final CorsConfiguration config = new CorsConfiguration();
            config.setAllowCredentials(true);
            config.setAllowedOrigins(Collections.singletonList("*"));
            config.setAllowedHeaders(Collections.singletonList("*"));
            config.setAllowedMethods(Arrays.stream(Http.HttpMethod.values()).map(Http.HttpMethod::name).collect(Collectors.toList()));
            source.registerCorsConfiguration("/**", config);
            return new CorsFilter(source);
    }

    ```
    - add and commit to github

2. Go back to the Ubuntu instance (ubuntu terminal)
3. Go to your backend project (if it's in the github and clone it)
4. Go to the wrapper directory and type `sudo docker-compose up`
5. Go to EC2 get the PUBLIC DNS IPV4 and add to your browser: 
    `http://ec2-35-173-178-66.compute-1.amazonaws.com:8080/users`

6. now back to the the front end portion edit the .env and replace the localhost for your EC2 IPV4 address:
    `http://ec2-35-173-178-66.compute-1.amazonaws.com:8080`

NOTE: if you run into errors, just copy the link into axios... 

7. type `npm build`
8. Sync the folder with the bucket: `aws s3 sync build/ s3://bucket-seca-bruno`

NOTE IF YOU GOT ACCESS ERROR:

Go to your bucket and choose overview and select upload: 
Choose upload and upload all the content from inside of your build directory

Once is done, choose properties and click on Static website hosting and you might see the link there, click on it and you will see your website hosting. 






