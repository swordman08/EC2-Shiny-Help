# EC2-Shiny-Help
Tutorial for classmates having trouble running there shiny app on EC2 instance.




Troubleshooting Help

Contact Decker Mecham (dmecham@chapman.edu) or slack me for any additional help.

I use a Windows computer, but most of this is applicable to both Mac and Windows; pathways will be different from person to person. All bash code is applicable to both Windows and Mac.

It is possible I made small typos in the commands, if the bash code does not run, just ask chat gpt what is wrong with the code snippet, it will probably be a spacing issue, I had some trouble copy and pasting a few things.

I will cover-
Authorization / granting permission
Python virtual environment setup.
Resizing volumes for added space that may be needed for xgboost install
Commonly used Bash Commands

Sorry for the Word Soup, If you take your time to read it all, you might find something of value (I hope you do, I spent a decent amount of time trying to cover everything I struggled with and found along the way).


To begin any of these tutorials SSH into your server.

Authorization / granting permission + virtual environment tip at the end of this section found at the "(DISCLAIMER)". (I recommended trying this method first rather than the other few ways to create virtual environments, where I ran into permission issues.) For the permissions section, just change the paths to the correct virtual environment and your app.

This bash or terminal command will recursively set the shiny user and group as the owner of all files and folders within shiny-app. If you called your folder something different, change shiny-app to whatever that is.

sudo chown -R shiny:shiny /srv/shiny-server/shiny-app

Afer that, this command sets permissions so that the shiny user can read, write, and execute files in shiny-app, while other users have read and execute permissions (necessary for web access).

sudo chmod -R 755 /srv/shiny-server/shiny-app


You can verify that the permissions are set correctly by listing the directory contents:

ls -l /srv/shiny-server/shiny-app

Each file and directory should now be owned by shiny:shiny and have permissions that look like this for directories:

drwxr-xr-x shiny shiny …

And for files

-rwxr-xr-x shiny shiny …

After adjusting permissions, restart Shiny Server to ensure it has access to the updated directory:

sudo systemctl restart shiny-server


For your virtual Python environment, check permissions like this. Path may vary.

ls -l /home/ubuntu/shiny_env

To allow shiny permission do the following command. Input the correct path as well.

sudo chown-R shiny:shiny /home/ubuntu/shiny_env

sudo chmod-R 755 /home/ubuntu/shiny_env

Than restart shiny server.

sudo systemctl restart shiny-server


Switch to the shiny user to test access to your app directory and files, especially the virtual environment and model file.

sudo -u shiny -s

While logged in as shiny, check for access to critical files:

ls -l /srv/shiny-server/Housing_Prices_Predictions/app.R
ls -l /srv/shiny-server/Housing_Prices_Predictions/real_estate_model.pkl
ls -l /home/ubuntu/shiny_env/bin/python




Try running R and loading the required libraries as the shiny user to confirm if the necessary environment is accessible: Run line by line

sudo R


library(reticulate)
use_python("/home/ubuntu/shiny_env/bin/python", required = TRUE)
joblib <- import("joblib")
model <- joblib$load("/srv/shiny-server/Housing_Prices_Predictions/real_estate_model.pkl")


Additional Troubleshooting Steps
Move the Virtual Environment to /srv/shiny-server
- Sometimes, having the virtual environment within the /home/ubuntu directory can cause access restrictions for other users.
- Try moving the virtual environment directly into /srv/shiny-server, where Shiny Server is configured to look for apps and related files by default.

sudo mv /home/ubuntu/shiny_env /srv/shiny-server/

Then adjust the permissions for the moved environment:

sudo chown -R shiny:shiny /srv/shiny-server/shiny_env
sudo chmod -R 755 /srv/shiny-server/shiny_env


In app.R, update the path to the virtual environment:


nano /srv/shiny-server/Housing_Prices_Predictions/app.R


library(reticulate)
use_python("/srv/shiny-server/shiny_env/bin/python",required =TRUE)


Test Access Again as shiny User
- After moving the environment, verify access again:

sudo -u shiny ls -l /srv/shiny-server/shiny_env/bin/python




If this doesn’t work, try this (this is what made my instance run) (below).

(DISCLAIMER) Start here if you are coming from the disclaimer below in the tutorial or if you are coming from above.

Since the shiny user doesn’t have permission to create or activate the environment, we’ll create it as root and then adjust permissions.

Create the Virtual Environment in a location that should be accessible,

Create the Virtual Environment in the shiny User’s Home Directory (this worked well for me)
- Since /srv/shiny-server is typically restricted, try creating the virtual environment in the shiny user’s home directory (e.g., /home/shiny/):

sudo -u shiny python3 -m venv /home/shiny/new_env/bin/python



Change Ownership of the Virtual Environment to shiny:

sudo chown -R shiny:shiny /home/shiny/new_env/bin/python

update app.R to Use the New Path

Point reticulate to this new environment in your app.R:

 Use this command to enter file
nano /srv/shiny-server/Housing_Prices_Predictions/app.R


library(reticulate)
use_python("/home/shiny/new_env/bin/python",required =TRUE)



Switch to the shiny User to install packages directly:

sudo -u shiny -s

Activate environment

source /home/shiny/new_env/bin/activate

Install packages

pip install xgboost joblib

Deactivate and exit Shiny user

deactivate
exit



Restart server

sudo systemctl restart shiny-server




Python Virtual Environment setup. Skip to (DISCLAIMER) which is above this. If that doesn’t work, you can try the following. But I advise you to try the disclaimer first.




Create a virtual environment in your home directory (or any preferred location):

python3 -m venv ~/shiny_env


Activate your virtual environment and install the packages you need:


source ~/shiny_env/bin/activate
 pip install xgboost joblib
 deactivate



In app.R, change the virtual environment path to:

Use this command to enter file       nano /srv/shiny-server/Housing_Prices_Predictions/app.R

library(reticulate)
use_virtualenv("/home/ubuntu/shiny_env",required =TRUE)



List the contents of the shiny_env directory to verify/confirm:

ls ~/shiny_env/bin/
If you see a python file there, the virtual environment was created correctly.

Restart server

sudo systemctl restart shiny-server


Ensure the virtual environment contains the required Python packages (xgboost and joblib) and is accessible.

Activate environment.


source ~/shiny_env/bin/activate

List packages installed on environment.

pip list

Deactivate after confirming or redo the above step to install xgboost and joblib.

deactivate


Test the virtual environment by opening R

sudo R


Run this line by line, using the appropriate file name of your saved xgboost pickel file. Don’t include ggplot2 if you don't use this.

library(shiny)
library(ggplot2)

library(reticulate)
use_python("/home/ubuntu/shiny_env/bin/python", required = TRUE)
joblib <- import("joblib")
model <- joblib$load("/srv/shiny-server/Housing_Prices_Predictions/real_estate_model.pkl")

Restart server

sudo systemctl restart shiny-server


Check server status with this command

sudo systemctl status shiny-server




If you want to edit your file within the server you can use this command

nano /srv/shiny-server/Housing_Prices_Predictions/app.R

Follow the commands to save your work and exit. I believe for windows it CTRL + O to save/write file, than you press enter on the file. Than you press CTRL + X to exit file.

Restart shiny server

sudo systemctl restart shiny-server


You can also view/read the file by using this command

cat /srv/shiny-server/Housing_Prices_Predictions/app.R


If use_virtualenv() works without issues, it’s fine to use. If use_virtualenv() fails or triggers configuration issues (such as the prompt to create a new environment), try use_python() for a more direct approach.

Using use_python() can be more reliable because it explicitly points to the Python executable in your virtual environment. This avoids potential issues where reticulate may not recognize the virtual environment due to path conflicts or permissions.


Change your app.R Replace the use_virtualenv line with.

use_python("/home/ubuntu/shiny_env/bin/python", required = TRUE)



To verify if any part of app.R is being executed, try adding a simple log message right at the top of app.R. This can help confirm if Shiny Server is even reaching app.R.


cat("Starting app.R\n", file = "/tmp/app_startup.log")


Add logging messages at critical points in app.R to identify where it might be failing. For example:


cat("Loaded libraries\n", file = "/tmp/app_startup.log", append = TRUE)
library(shiny)
library(ggplot2)
cat("Loaded shiny and ggplot2\n", file = "/tmp/app_startup.log", append = TRUE)
library(reticulate)
cat("Loaded reticulate\n", file = "/tmp/app_startup.log", append = TRUE)

use_python("/home/ubuntu/shiny_env/bin/python", required = TRUE)
cat("Set Python environment\n", file = "/tmp/app_startup.log", append = TRUE)

py_config_output <- capture.output(py_config())
cat(py_config_output, file = "/tmp/app_startup.log", sep = "\n", append = TRUE)

Restart server

sudo systemctl restart shiny-server

Check logs

cat /tmp/app_startup.log

If no logs show up, your app didn’t even start. It may be a permissions issue.



Resizing Directories in AWS for Ec2 instance.

On the left side of your AWS page, you will see Dashboard, Instances, Images, Elastic Block Store, etc. And many other things.

Click on Elastic Block Store.

Once here, select your instance.

In the top right you will see "Actions", click this.

Click on Modify Volume (the first schoice in drop down list)

Select the volume as general purpose SSD (gp3)

Modify the size to something like 12-14 gb. The Xgboost will only use around a gigabyte I believe, but give room for a few gigabytes atleast.

Don’t change anything else and click modify.

Reboot your server. With either AWS site or through the SSH via the command

sudo reboot

Check the added disk size by doing

df -h



Final Notes:

Check Permissions always

ls-l /srv/shiny-server/shiny-app

Make sure shiny-shiny has access.

Each folder should look like
drwxr-xr-x shiny shiny …

And file

-rwxr-xr-x shiny shiny ...


Use this to remove any directory not needed anymore. And Remember, you do not want any other R files in your directory besides app.R, having the server.R and ui.R will cause errors I believe. You must combine the two into one "app.R" file.

sudo rm-r /path/to/directory


Ensure the virtual environment contains the required Python packages (xgboost and joblib, or whatever you need) and is accessible.

Activate environment. (put whatever the path is to your python environment)


source ~/shiny_env/bin/activate

List packages installed on environment.

pip list

If a package is missing, like xgboost. Redo the steps above in tutorial to install it. I ran into an issue before where I


If you have unfixable errors like a bad environment where you can't install xgboost or you don’t have access due to where it's stored. You can delete that bad environment with

Remove the problematic new_env environment and recreate it from scratch.

rm -rf /srv/shiny-server/new_env


Recreate the environment using the python3 -m venv command to ensure it is fully independent of the system environment:

python3 -m venv /srv/shiny-server/new_env

Import Note

My final product uses python in the following path.

use_python("/home/shiny/new_env/bin/python", required = TRUE), you may find that your program runs once you follow the steps in the DISCLAIMER above. I didn't mean to make the tutorial so confusing, but it involves a lot of debugging and hours and hours of trial and error. I couldn't be perfectly precise, but it will have to do. It would be worth it if I could help a single person with a single problem.


Be kind to yourselves. This is a hard process. Many of us are doing something like this for the first time. Errors are inevitable. Stick with the problem until you find a solution. You will feel amazing once you solve the problem.




Helpful overview of the commonly used commands. Have fun !!!

# Setting Permissions for Shiny Applications and Virtual Environments

# Set ownership for Shiny app files
sudo chown -R shiny:shiny /path/to/shiny-app

# Set file permissions
sudo chmod -R 755 /path/to/shiny-app

# Restart Shiny Server to apply changes
sudo systemctl restart shiny-server

# Set permissions for the virtual environment
sudo chown -R shiny:shiny /path/to/shiny_env

sudo chmod -R 755 /path/to/shiny_env

# Checking Permissions and Testing Access as Shiny User
# Switch to the `shiny` user to test access
sudo -u shiny -s

# List critical files to verify access
ls -l /path/to/shiny-app/app.R

ls -l /path/to/shiny_env/bin/python

# Load libraries in R (as `shiny` user)
sudo R

library(reticulate)

use_python("/path/to/shiny_env/bin/python", required = TRUE)

# Moving the Virtual Environment
# Move the virtual environment
sudo mv /original/path/to/shiny_env /new/path/to/shiny_env

# Update permissions for the new location
sudo chown -R shiny:shiny /new/path/to/shiny_env

sudo chmod -R 755 /new/path/to/shiny_env

# Update app.R to reflect the new path
# Edit the R script app.R
nano /path/to/shiny-app/app.R
# Inside app.R, add or edit the line
use_python("/new/path/to/shiny_env/bin/python", required = TRUE)

# Creating and Setting Up a Python Virtual Environment
# Create a new virtual environment in the desired directory
sudo -u shiny python3 -m venv /path/to/new_env

# Activate and install required packages
source /path/to/new_env/bin/activate

pip install xgboost joblib

deactivate

# Update app.R to use the new environment

# Edit the R script app.R
nano /path/to/shiny-app/app.R
# Inside app.R, add or edit the line
use_python("/path/to/new_env/bin/python", required = TRUE)

# Troubleshooting and Logging

# Log messages in app.R to debug

# Edit the R script app.R
nano /path/to/shiny-app/app.R
# Inside app.R, add these lines to create a log for debugging
cat("Starting app.R\n", file = "/tmp/app_startup.log")

# Check the log file to verify execution steps
cat /tmp/app_startup.log

# Resizing EC2 Volume (if needed for space)
# Go to Elastic Block Store > Modify Volume in AWS Console and increase the size.
# Reboot the instance to apply changes
sudo reboot

# Additional Commands
# Check current permissions
ls -l /path/to/shiny-app

# Delete a problematic directory
sudo rm -rf /path/to/bad-directory

# Recreate the virtual environment from scratch if necessary
python3 -m venv /path/to/new_env

#check disk space in instance

df -h

