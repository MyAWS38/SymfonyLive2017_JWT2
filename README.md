# SymfonyLive2017_JWT2
AbstractGuardAuthenticator


Implementing JWT Authentication to your API Platform application
Go to the profile of Emir Karşıyakalı
Emir Karşıyakalı
Apr 29, 2018

API Platform allows to easily add a JWT-based authentication to your API using LexikJWTAuthenticationBundle. This bundle provides JWT(JSON Web Token) authentication for your Symfony API. But there's no official documentation for Symfony 4 (w/Flex) yet. This is a quick manual for implementing LexikJWTAuthenticationBundle.
1. User Provider

According to the API Platform documentation:

    LexikJWTAuthenticationBundle requires your application to have a properly configured user provider. You can either use the Doctrine user provider provided by Symfony (recommended), create a custom user provider or use API Platform's FOSUserBundle integration.

I decided to go with Doctrine User Provider. Because of Security component comes with API Platform I only need to create an entity named User which implements UserInterface or AdvancedUserInterface:
src/Entities/User.php

$ php bin/console doctrine:database:create
$ php bin/console doctrine:schema:update --force

2. Add JWTAuthentcationBundle

We need to add lexik/jwt-authentication-bundle to our composer.json file:

composer req "lexik/jwt-authentication-bundle"

    Thanks to Symfony Flex we don't need to add bundle definition it's already loaded from recipe.

2.1: Generate the SSH keys

$ mkdir config/jwt
$ openssl genrsa -out config/jwt/private.pem -aes256 4096
$ openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem

In case first openssl command forces you to input password use following to get the private key decrypted

$ openssl rsa -in config/jwt/private.pem -out config/jwt/private2.pem
$ mv config/jwt/private.pem config/jwt/private.pem-back
$ mv config/jwt/private2.pem config/jwt/private.pem

2.2: Handlers

We need to create Routes and Controller that responsible for handling Registration and Login:
config/routes.yml
src/Controller/AuthController.php
2.3: Security System

Lastly we need to tell Symfony’s security system about our Provider and Authenticator:
config/packages/security.yaml
Usage
Register a new user:

$ curl -X POST http://localhost:8000/register -d _username=johndoe -d _password=test
-> User johndoe successfully created

Get a JWT Token:

$ curl -X POST -H "Content-Type: application/json" http://localhost:8000/login_check -d '{"username":"johndoe","password":"test"}'
-> { "token": "[TOKEN]" }

Example of accessing secured routes:

$ curl -H "Authorization: Bearer [TOKEN]" http://localhost:8000/api
-> Logged in as johndoe

That's all folks! For the testing, API Platform documentation has a part for Behat and implementing @login annotation to automatically add a valid Authorization header, and with @logout to be sure to destroy the token after this scenario., here.
