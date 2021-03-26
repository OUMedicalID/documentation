<p align="center">
  <img src="https://i.imgur.com/frzDe3U.png" alt="Sublime's custom image" width="300"/>
</p>

# Medial ID
Medical ID is our capstone project created during the Winter 2021 semester. Medical ID's purpose is to make it easier and secure to transfer confidential patient information to doctors offices and other medical institutions through the use of Near Field Communication (NFC). 

Users will simply go near a Raspberry Pi and activate NFC to have their information quickly and securely collected rather than having to film out a large form.

# Technologies Used
To demonstrate sufficient knowledge in Computer Science and Information Technology, this project utilized many technologies and programming languages. Below is a list of all technologies and programming languages used.

 - **Front End Development**: HTML, CSS, JavaScript, Bootstrap, React
 - **Back End Development**: PHP
 - **Mobile Application Development**: iOS (Swift) and Android (Java)
  - **Databases**: SQL (mySQL software)
  - **Operating Systems**: Debian and Raspbian (Linux distributions)
  - **Other Programming Languages**: Python
  - **Other Knowledge**: Deploying a virtual private server, running a web server behind CloudFlare, configuring a [LAMP](https://www.ibm.com/cloud/learn/lamp-stack-explained) stack,AES encryption and good security practices

# Mobile Applications

A large part of this project centers around mobile applications for the iOS and Android platforms. The purpose of the applications is to securely collect the users' information and store it securely. 

Users are able to fill out basic personal information such as name, gender, address, phone number, etc. as well as other items such as marital status, weight, height, ethnicity and insurance information. In addition to this, users can fill out emergency contact information and enter as much information as needed about their medical conditions and any injury information.

Because all user information is housed in the mobile applications, we designed the apps in such a way that maximum security is enforced. We go over this more in the next section below. 


# Mobile Application Security

When users save their information, it is saved in encrypted form in SharedPreferences (Android) with the key residing in the KeyStore or in a plist file on iOS (UserDefaults) with the key residing in the iOS keychain. In addition to this, all encrypted information is saved on a web server without the key. All encryption is performed with **AES-256-CBC**

Perhaps one of our biggest achievements is the fact that these mobile applications can successfully operate without the need of a password column in our web server's mySQL database. 

The way this works is that when a user first registers, we take the 
**SHA512** of their password and use that as the key to encrypt the email. From here, this encrypted email string is used as the primary key in the database and serves as an efficient primary key at that. It is not possible for another user to  generate the same string unless the email and password are the same. 

![Flow chart of login and register](https://i.imgur.com/huzJ0To.jpg)

Below is some psuedocode of the register functionality:

    function register(email, password){
	    pass_hash = sha512(password);
	    //AES.encrypt(string, key);
	    enc_email = AES.encrypt(email, pass_hash);
	    MedicalID.register(enc_email); 
	} 


When a user wants to login, the process is fairly similar. First the entered password is hashed with **SHA512** and is used as the key to encrypt the email. From there an API request is sent that fetches all of the users encrypted information and restores them to the application. From there, the applications attempts to decrypt all of the information that was fetched by utilizing the same sha512-hashed password used to perform the login. This should be the right key as the only way to fetch the correct record was through having a correctly encrypted email that relies on the user's password. 

**In a highly unlikely event where another users' encrypted information is returned by the server, it will be completely useless as such information could never be decrypted without the right key.** 

There is no way for anyone to decrypt the information without the right password. **Furthermore, in the worst case scenario where the database is breached, all of the encrypted data will be useless to a hacker.  The beauty of using an implementation where we don't even store the password in any shape or form is that hackers cannot try to crack a user's password through iterative brute-force or through rainbow tables.**

Below is a sample row of what our mySQL database looks like:
|email|MID_Name|MID_Birthday|
|--|--|--|
|30AE621F292899FFD791DE2F8F8A3DE6AEFC83BA047DE5388CDD6D1238546FB9|E5D3A942AA1647F8DD613A9164D16865:21EA1E2164C60BD0|5E166387DD674F6FAAAB791BC6891F05:7F9CB5C60D6E4B9C|

Aside from the email, all other rows are in the following format :
> `encryptedString:randomIV`


# HIPAA compliance

It is important for any application that deals with sensitive medical information to be HIPAA compliant. Below are some items we implemented to get towards HIPAA complaince:

 - All user information is encrypted with **AES-256-CBC**
 - All username/password combinations guaranteed to be unique.
 - Strong password requirements 
 - Preventing use of common passwords like 12345
 - Web server forces HTTPS and uses [HSTS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)  through CloudFlare
 - Records maintained of when information is accessed.
 - Records of last activity for each user.
 - A server daemon to delete accounts of users with months of inactivity.
 - Secondary authentication on apps through 2nd password or biometrics.


 

# NFC Setup
To be done later....

# Raspberry Pi Web Server
To be done later...

# Ways To Improve and Future Ideas 

To be done later...
