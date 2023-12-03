# Deploying a MERN Stack App on AWS with MongoDB Atlas, Nginx, Namecheap, and Certbot

## Note 1: Ensure your MongoDB is hosted on MongoDB Atlas.

##Note 2: In your package.son the build command should have GENERATE_SOURCEMAP=false

```bash
"scripts": {
        "build": "GENERATE_SOURCEMAP=false react-scripts build",
    },
```

##Note 3: Your react app should have a app.config.json
app.config.json which should contain the following
```bash
{
  apps : [
    {
      name      : "react-app",
      script    : "npx",
      interpreter: "none",
      args: "serve -s build -p <port_number>"
    }
  ]
}
```

### 1. Create an AWS Account and Launch EC2 Instance

1.1. Create an account with [AWS](https://aws.amazon.com/) and navigate to the EC2 dashboard.

1.2. Launch an instance:

1.3. Give your instance a name and select "Ubuntu" as the Amazon Machine Image (AMI).

1.4. Create a Key Pair and download the `.pem` file to a designated folder for key pair files.

1.5. Configure security groups to allow SSH, HTTP, and HTTPS traffic.

1.6. Set the Root volume to gp3 and configure additional settings if needed.

1.7. Launch the instance.

1.8. Navigate to the Instances dashboard to ensure your instance is running.

### 2. Allocate and Associate Elastic IP

2.1. In the sidebar, go to "Network and Security" and click on "Elastic IPs."

2.2. Allocate Elastic IP address:
 ```
 Actions > Allocate Elastic IP address
 ```

2.3. Add a new tag (Key: name, Value: ... EIP).

2.4. Select the allocated EIP, click on "Actions," and then "Associate Elastic IP address."

2.5. In the associate EIP dialog, select the instance from the drop-down and click "Associate."

2.6. Confirm that the instance's public IP now reflects the EIP.

### 3. Connect to the Instance

3.1. Click on the instance in the dashboard, then click "Connect."

3.2. In your terminal, navigate to the folder containing the key pair files:
 ```
 cd <folder_of_key_pair_files>
 ```

3.3. Set appropriate permissions for the `.pem` file:
 ```
 chmod 400 <key.pem>
 ```

3.4. Copy the example command provided in the AWS console and run:
 ```
 ssh -i <key.pem> ubuntu@<ip-address> -v
 ```

### 4. Update and Upgrade Linux Machine, Install Node and NVM and project installation

4.1. Update and upgrade the Linux machine and install necessary tools:
 ```bash
 sudo apt update
 sudo apt upgrade
 sudo apt install -y git htop wget
 ```

4.2. Install Node using NVM:
 ```bash
 curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
 # or
 wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
 ```

4.3. Set up NVM environment variables:
 ```bash
 export NVM_DIR="$HOME/.nvm"
 [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
 [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
 ```

4.4. Verify that NVM has been installed:
 ```bash
 nvm --version
 ```

4.5. Install the latest LTS version of Node:
 ```bash
 nvm install --lts
 ```

4.6. Check the Node version:
 ```bash
 node --version
 ```

4.7. Clone your Node.js server repository:
 ```bash
 cd /home/ubuntu
 git clone https://github.com/your-repository
 ```

4.8. Change into the repository directory:
 ```bash
 cd /home/ubuntu/your-repository
 ```

4.9. If there is a `.env` file needed, run:
 ```bash
 nano .env
 ```

4.10. Paste the content of the .env into the file, then press Ctrl+X, followed by Ctrl+Y, and Enter.

4.11. Install project dependencies:
 ```bash
 npm install
 ```

4.12. Run the server:
 ```bash
 npm start
#or
npm run dev whatever it is
 ```

4.13. In AWS, go to "Network & Security" and click on "Security Groups."

4.14. Select the most recent launch-wizard, click on "Inbound Rules," and then "Edit Inbound Rules."

4.15. Add a rule for your localhost port (e.g., 3000) and set the source to "anywhere." Save the rules.

4.16. Go to "Instances," copy the public IP (which is also the EIP), and paste it in your browser's search bar with the port (e.g., `public_IP:3000`). Your server should be running.

4.17. Install PM2:
 ```bash
 npm install -g pm2
 ```

4.18. Start the app with PM2 (run Node.js and the React App in the background and auto-restart on server restart):
 ```bash
 pm2 start index.js &&  pm2 save //In your backend folder
 pm2 start app.config.json &&  pm2 save//In your frontend folder
 ```

4.19. If you want PM2 to start on system boot, run:
 ```bash
 pm2 startup
 ```
### 5. Nginx and CertBot

5.1. Install Nginx:
 ```bash
 sudo apt install nginx
 ```
5.2. Edit the Nginx configuration file:
 ```bash
 sudo nano /etc/nginx/sites-available/default
 ```

5.3. Set the `server_name` to your domain (e.g., example.com www.example.com).
```bash
server_name example.com www.example.com;
```

5.4. Add a location block for API proxy:
 ```bash
location / {
        proxy_pass http://localhost:<port_number>; //Your frontend server localhost
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        }

 location /api {
     proxy_pass http://localhost:<port_number>; //Your backend server localhost
     proxy_http_version 1.1;
     proxy_set_header Upgrade $http_upgrade;
     proxy_set_header Connection 'upgrade';
     proxy_set_header Host $host;
     proxy_cache_bypass $http_upgrade;
 }
 ```

5.5. Save the changes by pressing Ctrl+X, followed by Ctrl+Y, and then Enter.

5.6. Run
 ```bash
 sudo service nginx restart
 ```

5.7. Go to Namecheap's DNS settings for your domain, add two records with the hosts set to `www` and `@`, the IP sections set to your public IPv4 in AWS (EIP).

5.8. Visit your domain (e.g., example.com or www.example.com), and everything should be running.

At this point, your application is running over HTTP. The next steps will cover setting up HTTPS.

5.12. To make your site secure run
```bash
sudo apt install certbot python3-certbot-nginx
#then
sudo certbot --nginx -d example.com -d www.example.com
```
