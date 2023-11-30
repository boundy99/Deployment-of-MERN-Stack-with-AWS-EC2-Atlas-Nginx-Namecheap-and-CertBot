# Deploying a MERN Stack App on AWS with MongoDB Atlas, Nginx, Namecheap, and Certbot

## Step 1: AWS Instance Creation

### 1.1 Create an AWS Account and Launch EC2 Instance

1. Create an account with [AWS](https://aws.amazon.com/) and navigate to the EC2 dashboard.

2. Launch an instance:


3. Give your instance a name and select "Ubuntu" as the Amazon Machine Image (AMI).

4. Create a Key Pair and download the `.pem` file to a designated folder for key pair files.

5. Configure security groups to allow SSH, HTTP, and HTTPS traffic.

6. Set the Root volume to gp3 and configure additional settings if needed.

7. Launch the instance.

8. Navigate to the Instances dashboard to ensure your instance is running.

### 1.2 Allocate and Associate Elastic IP

9. In the sidebar, go to "Network and Security" and click on "Elastic IPs."

10. Allocate Elastic IP address:
 ```
 Actions > Allocate Elastic IP address
 ```

11. Add a new tag (Key: name, Value: ... EIP).

12. Select the allocated EIP, click on "Actions," and then "Associate Elastic IP address."

13. In the associate EIP dialog, select the instance from the drop-down and click "Associate."

14. Confirm that the instance's public IP now reflects the EIP.

### 1.3 Connect to the Instance

15. Click on the instance in the dashboard, then click "Connect."

16. In your terminal, navigate to the folder containing the key pair files:
 ```
 cd <folder_of_key_pair_files>
 ```

17. Set appropriate permissions for the `.pem` file:
 ```
 chmod 400 <key.pem>
 ```

18. Copy the example command provided in the AWS console and run:
 ```
 ssh -i <key.pem> ubuntu@<ip-address> -v
 ```

### 1.4 Update and Upgrade Linux Machine and Install Node and NVM

19. Update and upgrade the Linux machine and install necessary tools:
 ```bash
 sudo apt update
 sudo apt upgrade
 sudo apt install -y git htop wget
 ```

20. Install Node using NVM:
 ```bash
 curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
 # or
 wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
 ```

21. Set up NVM environment variables:
 ```bash
 export NVM_DIR="$HOME/.nvm"
 [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
 [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
 ```

22. Verify that NVM has been installed:
 ```bash
 nvm --version
 ```

23. Install the latest LTS version of Node:
 ```bash
 nvm install --lts
 ```

24. Check the Node version:
 ```bash
 node --version
 ```

25. Clone your Node.js server repository:
 ```bash
 cd /home/ubuntu
 git clone https://github.com/your-repository
 ```

26. Change into the repository directory:
 ```bash
 cd /home/ubuntu/your-repository
 ```

27. If there is a `.env` file needed, run:
 ```bash
 nano .env
 ```

28. Paste the content of the .env into the file, then press Ctrl+X, followed by Ctrl+Y, and Enter.

29. Install project dependencies:
 ```bash
 npm install
 ```

30. Run the server:
 ```bash
 npm start
#or
npm run dev whatever it is
 ```

31. In AWS, go to "Network & Security" and click on "Security Groups."

32. Select the most recent launch-wizard, click on "Inbound Rules," and then "Edit Inbound Rules."

33. Add a rule for your localhost port (e.g., 3000) and set the source to "anywhere." Save the rules.

34. Go to "Instances," copy the public IP (which is also the EIP), and paste it in your browser's search bar with the port (e.g., `public_IP:3000`). Your server should be running.

35. Install PM2:
 ```bash
 npm install -g pm2
 ```

36. Start the app with PM2 (run Node.js in the background and auto-restart on server restart):
 ```bash
 pm2 start index.js //In your backend folder
 pm2 save
 ```

37. If you want PM2 to start on system boot, run:
 ```bash
 pm2 startup
 ```

38. Install Nginx:
 ```bash
 sudo apt install nginx
 ```

39. Assuming you have the build folder in your frontend, run:
 ```bash
 cd /var/www/
 mkdir microclient # May require sudo
 sudo chown -R ubuntu microclient/
 ```

40. Copy the build folder into the `microclient` folder:
 ```bash
 sudo cp -R ~/your-repository/pathTo/build/* /var/www/microclient
 ```

41. Edit the Nginx configuration file:
 ```bash
 sudo nano /etc/nginx/sites-available/default
 ```

42. Change `root /var/www;` to `root /var/www/microclient;`.

43. Set the `server_name` to your domain (e.g., example.com www.example.com).

44. Add a location block for API proxy:
 ```nginx
 location /api {
     proxy_pass http://localhost:3000; #Your backend server localhost
     proxy_http_version 1.1;
     proxy_set_header Upgrade $http_upgrade;
     proxy_set_header Connection 'upgrade';
     proxy_set_header Host $host;
     proxy_cache_bypass $http_upgrade;
 }
 ```

45. So far so good I hope hahahahha

46. Save the changes by pressing Ctrl+X, followed by Y, and then Enter.

47. Go to Namecheap's DNS settings for your domain, add a record with the host set to `www` and the IP to your public IPv4 in AWS (EIP).

48. Visit your domain (e.g., example.com), and everything should be running.

49. Ensure your MongoDB is hosted on MongoDB Atlas.

At this point, your application is running over HTTP. The next steps will cover setting up HTTPS.

---

Continue to the next sections for configuring HTTPS with Certbot.
