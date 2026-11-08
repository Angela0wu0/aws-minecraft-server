# How-to-Guide: Deploying a Minecraft Server on AWS
## Part 1: Setting Up The EC2 Instance
### Locate the EC2 Instance: 
-  Search and select for the EC2 in the AWS Console search bar.
-  Locate and click the **yellow button** named `Launch Instance`.

### Configuring the EC2 Instance for Minecraft:
1. Name and tags:
     - **Name:** `Minecraft Server`
    
2. Application and OS Images (Amazon Machine Images):
     - **AMI:** `Ubuntu Server 24.04 LTS (HVM), SSD Volume Type (Free Tier)`
    
3. [Instance type:](https://aws.amazon.com/blogs/gametech/cost-optimize-your-minecraft-java-ec2-server/)
    - ```t3.medium``` (0.0416 USD per hour)

4. Key pair:
    - Create a new key pair:
      - **Name:** `mcServerKey`
      - **Key Pair Type:** `RSA`
      - **Private Key File Format:** `.pem`
>[!IMPORTANT]
**Store the Private Key file in a safe area**
        
5. Network Settings:
    - Click **edit**
    - **Auto-assign public IP:** `enable`
    - **Firewall (security groups):** **Select `Create Security group`**
    - **Security group name:** `mcServerSG`
    - **Description:** `Allow SSH, TCP, and UDP for the Minecraft Server.`
      
6.[ Firewall (Security Group Rules):](https://localxpose.io/blog/minecraft-port-forwarding
)
>[!IMPORTANT]
**Remove any pre-existing Security group rule**
  - Add SSH security group rule:
       - **Type:** `SSH`
       - **Source Type:** `Anywhere`
         
  - Add Custom TCP security group rule:
       - Type: `Custom TCP` 
       - Port Range: `25565`
       - Source Type: `Anywhere`
       - Description: `Minecraft Java`
         
  - Add Custom UDP security group rule:
       - Type: `Custom UDP` 
       - Port Range: `19132-19133` 
       - Source Type: `Anywhere`
       - Description: `Minecraft Bedrock Edition IPv4 & IPv6`

 ### Launch the Instance
  - Wait for instance state to turn to `running`.
    
## Part 2: Creating an Elastic IP
   - This would keep the public IP static, then changing whenever the instance restarts.

### Locate the Elastic IPs: 
   - On the EC2 Dashboard on the left, locate `Elastic IPs`.

### Creating Elastic IPs
   - Click the yellow button named `Allocate Elastic IP address`.
   - Then click `Allocate`.
     
### Associating the Elastic IP with the instance
   - Select the created Elastic IP address and click on `Actions`.
   - Then `Associate Elastic IP address`.
   - Choose the `Minecraft Server` instance.
   - Finish by clicking `Associate`.
     
## Part 3: Installing Minecraft Server Dependencies
### Connect To The Created Instance
   - If on Mac/Linux, open your terminal, open PowerShell if on Windows.
   - Locate your key file (`mcServerKey.pem`).
   - Then run the following SSH command:
> **Note:** Replace `<Public-IP>` with the instance's Public IPv4 IP
     
**Bash**
```
chmod 400 mcServerKey.pem
ssh -i mcServerKey.pem ubuntu@<Public-IP>
```
### Setting Up the Server
1. Updating and Installing Dependencies:
    - We will be using the [Long-Term Support (LTS) Release version 25](https://www.oracle.com/java/technologies/downloads/) of the Java Development Kit (JDK)
    
     **Bash**
     ```
     sudo apt update -y
     sudo apt install openjdk-25-jre-headless -y
     ```

2. Make and Access the `minecraft-server` Folder
   
     **Bash**
     ```
     mkdir ~/minecraft-server && cd ~/minecraft-server
     ```
4. Download the Minecraft Server File Into the Created Folder  

     **Bash**
    ```
    wget -O server.jar "$(curl -s "$(curl -s https://launchermeta.mojang.com/mc/game/version_manifest.json | jq -r      '.versions[] | select(.id=="26.1.2") | .url')" | jq -r '.downloads.server.url')"
    ```
  
   
6. [Start the Minecraft Server ](https://www.minecraft.net/en-us/download/server)
     - First, run the Server
       
     **Bash**
    ```
    java -Xmx2G -Xms2G -jar server.jar nogui
    ```
    

8. Agree to the EULA to Run the Server
     - Access the `eula.txt` file:
       
     **Bash**
     ```
     vim eula.txt
     ```
     
     - Press `i` on the keyboard and the bottom left of the terminal should now say `INSERT`.
     - Replace `False` with `True`
     - Then hit `Esc` and type `:` using `shift + ;`.
     - Then type `wq` and hit `Enter`.
     - Now run the Server again:

       **Bash**
       ```
       java -Xmx2G -Xms2G -jar server.jar nogui
       ```
     > ***The Minecraft Server should now be Active***

## Part 4: Set up auto-start
### Create and configure the service for Minecraft:
      
 **Bash**
 ```
 sudo vim /etc/systemd/system/minecraft.service
 ```
   - Press `i` on the keyboard, and the bottom left of the terminal should now say `INSERT`.
   - Copy the below Service Configuration into the file: 
     ```
     [Unit]
     Description=Minecraft Server
     After=network.target
     [Service]
     User=ubuntu
     WorkingDirectory=/home/ubuntu/minecraft-server
     ExecStart=/usr/bin/java -Xmx2G -Xms2G -jar server.jar nogui
     Restart=on-failure
     [Install]
     WantedBy = multi-user.target
     ```
- Then hit `Esc` and type `:` using `shift + ;`.
- Then type `wq` and hit `Enter`.
  
### Check if the Service Configuration is copied over correctly 
- The Minecraft Service should now be created. To make sure the contents were properly saved, use the following command:

  **Bash**
  ```
  cat /etc/systemd/system/minecraft.service
  ```

  > **IMPORTANT: Make sure the Service Configuration is copied over correctly**

5. Enable and Start the Minecraft Service

   **Bash**
     ```
     sudo systemctl enable minecraft
     sudo systemctl start minecraft
     sudo systemctl status minecraft
     ```
## Part 5: Access the Minecraft Server
### Initial Setup 
   - Download [Minecraft](https://www.minecraft.net/en-us/download)
   - Open the Minecraft Launcher and Sign In
   - Click on Minecraft Java Edition on the left-hand side of the Minecraft Launcher
   - Click **Play**
### Connecting to the Server
   - Click on `Multiplayer`
   - Then `Add Server`:
      - Server Name: `Minecraft Server`
      - Server Address: `<Public-IP>`
       > **Note:** Replace `<Public-IP>` with the instance's Public IPv4 IP

          
                  
            



