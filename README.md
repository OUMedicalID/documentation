# Medial ID

Medical ID is our capstone project created during the Winter 2021 semester. Medical ID's purpose is to make it easier and secure to transfer confidential patient information to doctors offices and other medical institutions through the use of Near Field Communication (NFC)

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


When a user wants to login, the process is fairly similar. First the entered password is hashed with **SHA512** and is used as the key to encrypt the email. From there an API request is sent that fetches all of the users encrypted information and restores them to the application. From there, the applications attempts to decrypt all of the information that was fetched by utilizing the same sha512-hashed password used to perform the login. This should be the right key as the only way to fetch the correct record was through having a correctly encrypted email that relies on the user's password. **In a highly unlikely event where another users' encrypted information is returned by the server, it will be completely useless as such information could never be decrypted without the right key.** There is no way for anyone to decrypt the information without the right password. Furthermore, in the worst case scenario where the database is breached, all of the encrypted data will be useless to a hacker. **The beauty of using an implementation where we don't even store the password in any shape or form is that hackers cannot try to crack a user's password through iterative brute-force or through rainbow tables.**
