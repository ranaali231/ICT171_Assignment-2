# ICT171_Assignment-2
**Name:** Muhammad Ali  
**Domain:** [https://foodone.xyz](https://foodone.xyz)  
**IP:** 13.239.237.8

**Week 1:** Launching the Server and Initial Setup.
In week one, I started with creating an AWS account via AWS Educate and learning how to use AWS Management Console. I explored different services, then EC2 and created a virtual machine with Ubuntu Server 24.04 LTS, a stable, long term supported operating system. I opted for t2.micro, an eligible service under AWS’s free tier, with basic server settings. I created a new .pem key pair, one I would have to use on a secured basis. When creating the instance, I modified security group settings to permit incoming traffic on SSH (port 22), HTTP (port 80), and HTTPS (port 443). To ensure my instance was running, I then SSH connected into it by entering into where my .pem file was residing and entering SSH command AWS gave. I logged into my Ubuntu server. To get my environment ready, I updated system cached packages with sudo apt update then went ahead with Apache’s install with sudo apt install apache2. I checked for installation via a visit into my instance’s public IP address in a webpage, where I was welcomed with Apache’s welcome page a confirmation my web server was live and functional.

**Week 2:** Elastic IP and LAMP Stack Configuration.
Week 2 was spent making a public facing server ready for dynamic data. First, I assigned an Elastic IP (EIP) address to the instance. As a result, the public IP address no longer changed on reboot, losing no accessibility. Next, I went ahead with LAMP. PHP, required Apache PHP module, and PHP MySQL driver were installed via sudo apt install php libapache2 mod php php mysql. Next, I went ahead with the installation of MySQL server using sudo apt install mysql server. Post installation, I accessed MySQL as a root, set a different authentication plugin as mysql native password, and created a strong pass. Next, I created a new MySQL username (m_ali) as well as a database (m) solely for use by WordPress. Full permissions on the new database were assigned for this new username. This completed the server/config part, with the server ready for dynamic fetch/storing of data with WordPress.

**Week 3:** Deploying WordPress and Initial Configuration.
Week three, I deployed a complete WordPress site on my Apache server. I switched directories to /tmp on the server and downloaded a fresh version of the WordPress package with wget. I extracted the package using tar, then transferred the WordPress directory into Apache's document root at location /var/www/html/. There, I copied in the sample config file into wp-config.php and edited it with database credentials and modifications to get it working with MySQL. I then established proper permissions, accessed the site using the server’s public IP address with a suffix of /wordpress, which presented me with the WordPress install page. I completed the install using the browser-based interface, and as a precaution, also updated wp-config.php manually in case automatic file writing fails. That finished off the install, and I could then access the WordPress admin interface and begin basic customizations and config of content.

**Week 4:** Domain Integration and HTTPS Security.
I set up a custom domain and secured my site last week. I purchased a custom domain, named foodone.xyz, from Namecheap. I set A records in Namecheap’s control panel under Advanced DNS for mapping the root domain (@) and www subdomain to my EC2’s Elastic IP address. Back on my server, I edited Apache virtual host config in /etc/apache2/sites-available/000-default.conf to set a ServerName with my domain. I also made two additions in wp-config.php with WP_HOME & WP_SITEURL with my URL from a domain. Following an Apache restart as well as a wait for DNS propagation, my site was accessible over my domain. Lastly, I set up Certbot to activate HTTPS. Following a resolution of an initial repo issue, I was able to install Certbot with Apache plugin with ease, then executed sudo certbot --apache to obtain a free SSL certificate from Let’s Encrypt. The tool automatically configured Apache to serve HTTP on HTTPS. Testing with a visit in browsers using https://foodone.xyz presented me with a secure padlock icon—indicating that SSL was properly functional as well as my site was fully secured.





**Backup Script**

#!/bin/bash

WEB_DIR="/var/www/html/wordpress"
BACKUP_DIR="/home/ubuntu/website_backups"
DB_NAME="m_ali"
DB_USER="m_ali"
DB_PASS="pak@123"
DATE=$(date +"%Y-%m-%d_%H-%M")
LOG_FILE="$BACKUP_DIR/backup_log.txt"
DAYS_TO_KEEP=7

if [ ! -d "$BACKUP_DIR" ]; then
    mkdir "$BACKUP_DIR"
fi

BACKUP_PATH="$BACKUP_DIR/backup-$DATE"
mkdir "$BACKUP_PATH"

echo "Backup started on $(date)" >> "$LOG_FILE"

echo "Copying website files..." >> "$LOG_FILE"
tar -czf "$BACKUP_PATH/wordpress_files.tar.gz" "$WEB_DIR"

echo "Saving database..." >> "$LOG_FILE"
mysqldump -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" > "$BACKUP_PATH/db_backup.sql"

echo "Backup done! Saved in $BACKUP_PATH" >> "$LOG_FILE"

echo "Cleaning up old backups..." >> "$LOG_FILE"
find "$BACKUP_DIR" -maxdepth 1 -type d -name "backup-*" -mtime +$DAYS_TO_KEEP -exec rm -rf {} \; >> "$LOG_FILE"

echo "Finished on $(date)" >> "$LOG_FILE"
echo "------------------------------" >> "$LOG_FILE"


**Explanation**

The following is a backup script for backing up website files and MySQL database for a website using WordPress. It starts with a line declaring it uses Bash shell to run it. It sets a number of variables for use in the process. These include where website files are (/var/www/html/wordpress), where backups are stored (/home/ubuntu/website_backups), database name (m_ali), database username (m_ali), and password (pak@123). It also sets a variable as current date and time, which it uses as a prefix while naming backups so no backup would overwrite an older one. It uses a different variable as a location of a log file documenting script activity and yet a different one as how many days' backups should be retained (7 days in this script).

The script then searches for a backup directory, and if it does not already have one, it creates it. Creates a new folder for this specific backup with a date as well as timestamp. Logs the date and timestamp as a start time for record-keeping in a log file. Backs up the website files. Logs, compresses the entire folder containing WordPress into a single archive file (in .tar.gz form), then moves it into the new backup folder. Once the backup file is complete, the script diligently marks the beginning database backup before using the mysqldump command to export the contents of the WordPress database into an .sql file, also in a backup folder. Having backed up the database as well as site files, the script marks backup completion alongside where it has been stored.

To prevent disk space from filling up as well as declutter, the script also searches for older backups within the backup directory. It searches for any directories with a name in the format and more than 7 days old and removes them automatically. It furthermore marks as finished the time a process has completed as well as contains a divider line within its log file, separating it from other backup entries, making it more readable. The script as a whole is a great way of backing up a WordPress site on a regular basis as well as keeping backups organized.
