## Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg


```


## Add Docker’s official GPG key:

```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

```



## Use the following command to set up the repository:

```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


```


## Update the apt package index:

```
sudo apt-get update
```
  

## To install the latest version, run:
```
  sudo apt-get install docker-ce

```


## OPTIONAL
###  Executing the Docker Command Without Sudo

### If you want to avoid typing sudo whenever you run the docker command, add your username to the docker group:

```
sudo usermod -aG docker ${USER}

```



### To apply the new group membership, log out of the server and back in, or type the following:

```
su - ${USER}
```
