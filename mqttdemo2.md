## Instructions to prepare for mqtt demo

The counterpart for python program mqttdemo2.py  is mqttdemo2.html  


````bash
# install python package management software pip
sudo apt install python3-pip -y

# verify
pip3 --version

# install virtual environment support
sudo apt install python3-venv -y

# create virtualenvironment for our project
python3 -m venv mqttdemo

# go to mqttdemo
cd mqttdemo

# activate mqttdemo virtualenvironment
source bin/activate

# get the demo from my repository
wget "I give the link"

# Install package paho.mqtt
pip install paho.mqtt

# start the program
python3 mqttdemo.py

````



