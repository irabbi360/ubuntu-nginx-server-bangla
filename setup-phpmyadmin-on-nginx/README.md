# Setup PhpMyAdmin Ubuntu & Config with Nginx

## Step 1 - Update Your System
Access the command line and use the apt package manager to update the Ubuntu package repository and installed packages:

```bash
sudo apt update && sudo apt upgrade -y
```
Allow the operation to finish.


## Step 2 - Step 2: Install phpMyAdmin
1. Run the following command to install phpMyAdmin and its dependencies:
```bash
sudo apt install phpmyadmin -y
```
2. The installer prompts you to choose a web server to configure automatically. The apache2 option is already highlighted if the system uses Apache. Press `Space` to select apache2, then `Tab` to highlight `Ok`.
3. Press `Enter` to confirm the selection.
4. (Optional) Select `Yes` and `press` Enter to set up a phpMyAdmin database using the dbconfig-common configuration package.
5. The installer creates a default user named phpmyadmin. Type a strong password for the phpmyadmin user and hit `Enter`.