# Procedure to install Jitsi with JWT authentications. I have used Ubuntu 18.4 bionic, recommendation is try it in a new machine.

**1. Update ubuntu packages after fresh installation.**
 
    $sudo apt update

**2. Setup hostname**

    sudo hostnamectl set-hostname meet
  
  add the same name in /etc/hosts
    
    x.x.x.x meet.example.org
    
  Where x.x.x.x is your server's public IP address.
  
  Confirm you are able ping using hostname 
         
    ping "$(hostname)"
 
**3. Install lua 5.2**
    
    apt-get update -y &&
    apt-get install gcc -y &&
    apt-get install unzip -y &&
    apt-get install lua5.2 -y &&
    apt-get install liblua5.2 -y &&
    apt-get install luarocks -y &&
    luarocks install basexx &&
    apt-get install libssl1.0-dev -y &&
    luarocks install luacrypto 
  
 **4.Install Lua Cjson Library and luajwtjitsi. It provides JSON parsing and encoding support for Lua**
    
    mkdir cjson &&
    cd cjson &&
    luarocks download lua-cjson &&
    luarocks unpack lua-cjson-2.1.0.6-1.src.rock &&
    cd lua-cjson-2.1.0.6-1/lua-cjson &&
    sed -i 's/lua_objlen/lua_rawlen/g' lua_cjson.c &&
    sed -i 's|$(PREFIX)/include|/usr/include/lua5.2|g' Makefile &&
    luarocks make &&
    luarocks install luajwtjitsi &&
    cd
  
 **5. Install Prosody**
    
    wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add - &&
      echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list &&
      apt-get update -y &&
      apt-get upgrade -y &&
      apt-get install prosody -y 
   
   **6. Reboot your machine**
    
    shutdown -r now
   
   **7. Istall nginx**
    
    apt-get install nginx 
         
   **8. Install Jitsi and Jitsi jwt plugi**
    
    apt-get install nginx 
        wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add - &&
        sh -c "echo 'deb https://download.jitsi.org stable/' > /etc/apt/sources.list.d/jitsi-stable.list" &&
        apt-get -y update &&
        apt-get install jitsi-meet -y &&
        apt-get install jitsi-meet-tokens -y
    
  **9. Extra configuratons in /etc/prosody/prosody.cfg.lua** .
      Open /etc/prosody/prosody.cfg.lua and set following flag to false.
   
              c2s_require_encryption=false
      Make sure this line is present at the end of /etc/prosody/prosody.cfg.lua, if not please add .
           
           Include "conf.d/*.cfg.lua"
      Add these two lines , will be used to validate issue and audiance parameter inside jwt token.
        
          asap_accepted_issuers = { "jitsi", "shahid" }
          asap_accepted_audiences = { "jitsi", "shahid" }
        
   Note: Second value can be anything but should be matching with aud and iss paramter inside jwt.
     
   **10. Restart jitsi services.**
     
           systemctl restart prosody jicofo jitsi-videobridge2 jigasi
          
   **11. Forming JWT token**
        Go to https://jwt.io/#debugger-io website in brower.
        add following json content in Header part
         
              {
              "typ": "JWT",
               "alg": "HS256"
              }     
       add following json content in Payload part
        
      
     {
      "context": {
        "user": {
         "avatar": "https:/gravatar.com/avatar/abc123",
         "name": "shahidHUssain",
         "email": "shahid@example.com",
         "id": "abcd:a1b2c3-d4e5f6-0abc1-23de-abcdef01fedcba"
       }
      },
      "aud": "shahid",
      "iss": "shahid",
      "sub": "meet.example.org",
      "room": "*"
    }
    
  Enter your secret key at secret key column at bottom right corner field
    
 **12. Final Step. Copy the Encrypted hash content , form a URL in below format and paste in browser , enter :)**
        https://meet.example.org/angrywhalesgrowhigh?jwt=<Token>
        Exampe : 
  
          https://meet.example.org/angrywhalesgrowhigh?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJjb250ZXh0Ijp7InVzZXIiOnsiYXZhdGFyIjoiaHR0cHM6L2dyYXZhdGFyLmNvbS9hdmF0YXIvYWJjMTIzIiwibmFtZSI6InNoYWhpZEhVc3NhaW4iLCJlbWFpbCI6InNoYWhpZEBleGFtcGxlLmNvbSIsImlkIjoiYWJjZDphMWIyYzMtZDRlNWY2LTBhYmMxLTIzZGUtYWJjZGVmMDFmZWRjYmEifX0sImF1ZCI6InNoYWhpZCIsImlzcyI6InNoYWhpZCIsInN1YiI6Im1lZXQuZXhhbXBsZS5vcmciLCJyb29tIjoiKiJ9.Ffj-A459gx8KE1YFZW6nu0gstFLRgXCZAMgwcIufaMo
