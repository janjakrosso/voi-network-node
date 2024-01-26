# voi-network-node
```console
# Updates:
sudo apt update && sudo apt-get upgrade -y
sudo systemctl start unattended-upgrades && sudo systemctl enable unattended-upgrades
```
```console
#  İnstalling Algorand:
sudo apt install -y jq gnupg2 curl software-properties-common
curl -o - https://releases.algorand.com/key.pub | sudo tee /etc/apt/trusted.gpg.d/algorand.asc
# You can say ENTER to the output.
sudo add-apt-repository "deb [arch=amd64] https://releases.algorand.com/deb/ stable main"
# Let's update it again and stop the node from starting automatically:
sudo apt update && sudo apt install -y algorand && echo OK
sudo systemctl stop algorand && sudo systemctl disable algorand && echo OK
# let's make goal setup
echo -e "\nexport ALGORAND_DATA=/var/lib/algorand/" >> ~/.bashrc && source ~/.bashrc && echo OK
sudo adduser $(whoami) algorand && echo OK
# configuration
sudo algocfg set -p DNSBootstrapID -v "<network>.voi.network" -d /var/lib/algorand/ &&\
sudo algocfg set -p EnableCatchupFromArchiveServers -v true -d /var/lib/algorand/ &&\
sudo chown algorand:algorand /var/lib/algorand/config.json &&\
sudo chmod g+w /var/lib/algorand/config.json &&\
echo OK
# Genesis
sudo curl -s -o /var/lib/algorand/genesis.json https://testnet-api.voi.nodly.io/genesis &&\
sudo chown algorand:algorand /var/lib/algorand/genesis.json &&\
echo OK
```
```console
# Let's configure the algorithm as Voi:
sudo cp /lib/systemd/system/algorand.service /etc/systemd/system/voi.service &&\
sudo sed -i 's/Algorand daemon/Voi daemon/g' /etc/systemd/system/voi.service &&\
echo OK
# let's run nodeu:
sudo systemctl start voi && sudo systemctl enable voi && echo OK
# Let's check the node with status:
goal node status
# ==> Genesis ID: voitest-v1
# ==> Genesis hash: IXnoWtviVVJW5LGivNFc0Dq14V3kqaXuK2u5OQrdVZo=
# This should be the end of the output (hash may change)

# Let's sync fast:
goal node catchup $(curl -s https://testnet-api.voi.nodly.io/v2/status|jq -r '.["last-catchpoint"]') &&\
echo OK
# Let's wait here for a few minutes

# Let's do status again but this time we will see Catchpoint in the logs:
goal node status

# When we check with this command, let's wait for the Sync Time to reset and the Catchpoint to go in the logs.
goal node status -w 1000
# CTRL + C when the above conditions are met
```
```console
Edit the part that says # Test and remove the quotes <>
sudo ALGORAND_DATA=/var/lib/algorand diagcfg telemetry name -n <Test>

sudo ALGORAND_DATA=/var/lib/algorand diagcfg telemetry enable &&\
sudo systemctl restart voi
```
```console
# Let's create a wallet:
goal wallet new voi
# After you set a password, say Y and take your 24 words and store them.

# Now let's import our wallet into our node:
goal account import
# When we enter the password and 24 words it will give us an Imported address, let's keep this as our wallet address.

# Now let's enter these codes and it will ask us for our Imported address.
# You can copy and paste this code at once
echo -ne "\nEnter your voi address: " && read addr &&\
echo -ne "\nEnter duration in rounds [press ENTER to accept default (2M)]: " && read duration &&\
start=$(goal node status | grep "Last committed block:" | cut -d\  -f4) &&\
duration=${duration:-2000000} &&\
end=$((start + duration)) &&\
dilution=$(echo "sqrt($end - $start)" | bc) &&\
goal account addpartkey -a $addr --roundFirstValid $start --roundLastValid $end --keyDilution $dilution
#In the question after # Imported, we can say ENTER and choose the default.
# Wait for the import to complete and save your Participation ID.

# Let's look at our activity, it's out here!!! SHOULD BE OFFLINED!!!
checkonline() {
  if [ "$addr" == "" ]; then echo -ne "\nEnter your voi address: " && read addr; else echo ""; fi
  goal account dump -a $addr | jq -r 'if (.onl == 1) then "You are online!" else "You are offline." end'
}
checkonline
```
Get token from discord to continue from this stage. node-runners channel => /voi-testnet-faucet.
```console
# If we got our token, with this command !!Let's get online !!!
getaddress() {
  if [ "$addr" == "" ]; then echo -ne "\nEnter your voi address: " && read addr; else echo ""; fi
}
getaddress &&\
goal account changeonlinestatus -a $addr -o=1 &&\
sleep 1 &&\
goal account dump -a $addr | jq -r 'if (.onl == 1) then "You are online!" else "You are offline." end'
```

THAT'S ALL
