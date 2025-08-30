1.Setup EC2 & App
# Update server
sudo yum update -y

# Install required packages
sudo yum install -y git nodejs

# Clone your app
git clone https://github.com/kranthi619/My-NodeJS.git
mv My-NodeJS/blogify/* ~/myapp/

# Go into project folder
cd ~/myapp

# Initialize git repo (if needed)
git init
2. Install Dependencies & Run
# Install dependencies
npm install

# If package.json doesn’t include required modules, install explicitly
npm install express mongoose cookie-parser dotenv ejs

# Start the app
node app.js
# or (if you have npm start script)
npm start
3. Environment Variables

Edit .env file (MongoDB + PORT): vi .env
Example:
PORT=8000
MONGO_URL=mongodb+srv://<username>:<password>@cluster0.ncbjvwo.mongodb.net/mydb?retryWrites=true&w=majority
4. Keep App Running in Background
# Install pm2 process manager
sudo npm install -g pm2

# Start app with pm2
pm2 start app.js --name myapp

# Enable auto start on reboot
pm2 startup
pm2 save
5.useful tips
List running apps:pm2 list
Stop app by name (replace myapp with your app name shown in the list):pm2 stop myapp
Stop app by ID (replace 0 with the process ID from pm2 list):pm2 stop 0
Stop all apps at once:pm2 stop all
After stopping, you can check with:pm2 list

.....feature app upgradations.............
🔹 Case 1: You browse app through Load Balancer (LB)

When you attach an Application Load Balancer (ALB) or NLB, traffic comes to the LB DNS name (not EC2 public IPs).

Your app servers (EC2s) are behind the LB.

But MongoDB is not behind the LB → it’s a separate DB service (running on EC2 or Atlas or DocumentDB).

👉 For DB access, your application servers (private EC2s) must reach MongoDB.

If MongoDB is running on an EC2, instead of adding its public IP range in security group, you should:

Put MongoDB in a private subnet.

Allow access only from the App Server Security Group, not from “0.0.0.0/0” or public IPs.

This way, no matter how EC2s scale in/out, they will always be able to talk to MongoDB because the SG-to-SG rule allows it (not IP based).

🔹 Case 2: Auto Scaling increases EC2 servers

When EC2s scale up, new instances get new private IPs. If you allow DB access by IP range, it breaks because DB won’t recognize the new IP.

✅ Best Practice:

Use Security Groups (SGs), not IPs.

Example:

App EC2 SG = app-sg

DB EC2 SG = db-sg

Rule: db-sg allows inbound on 27017 from app-sg

Now, whenever Auto Scaling adds/removes EC2s, they are automatically in app-sg, so they always have DB access.

🔹 Recommended Production Setup

MongoDB:

Prefer a managed service (MongoDB Atlas or AWS DocumentDB) → handles scaling, backups, HA.

If self-hosted → run on separate EC2s in private subnet with replication for HA.

App EC2s:

Behind ALB.

In Auto Scaling Group.

Use SG-to-SG communication with DB.

LB DNS for frontend, DB SG for backend.

LB handles public traffic.

DB never exposed publicly.

⚡ Example Security Group Rule (best practice):

db-sg inbound: allow MongoDB port 27017 from app-sg (not IPs).


