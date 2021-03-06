# Creating Your Own Offline Voice Assistant Edge Service for Raspberry Pi

Follow the steps in this page to create your own Offline Voice Assistant Horizon edge service.

## Preconditions for Developing Your Own Service

1. If you have not already done so, complete the steps in these sections:

  - [Preconditions for Using the Offline Voice Assistant Example Edge Service](README.md#preconditions)
  - [Using the Offline Voice Assistant Example Edge Service with Deployment Pattern](README.md#using-processtext-pattern)

2. If you are using macOS as your development host, configure Docker to store credentials in `~/.docker`:

  - Open the Docker **Preferences** dialog
  - Uncheck **Securely store Docker logins in macOS keychain**

3. If you do not already have a docker ID, obtain one at https://hub.docker.com/ . Log in to Docker Hub using your Docker Hub ID:

  ```bash
  export DOCKER_HUB_ID="<dockerhubid>"
  echo "<dockerhubpassword>" | docker login -u $DOCKER_HUB_ID --password-stdin
  ```

  Output example:

  ```bash
  WARNING! Your password will be stored unencrypted in /home/pi/.docker/config.json.
  Configure a credential helper to remove this warning. See
  https://docs.docker.com/engine/reference/commandline/login/#credentials-store

  Login Succeeded
  ```

4. Create a cryptographic signing key pair. This enables you to sign services when publishing them to the exchange. **This step only needs to be done once.**

  ```bash
  hzn key create "<x509-org>" "<x509-cn>"
  ```

  where `<x509-org>` is your company name, and `<x509-cn>` is typically set to your email address.

5. Install `git` and `jq`:

  On **Linux**:

  ```bash
  sudo apt install -y git jq
  ```

  On **macOS**:

  ```bash
  brew install git jq
  ```

# <a id=build-publish-your-ova> Building and Publishing Your Own Version of the Offline Voice Assistant Edge Service

1. Clone this git repo:
```bash
cd ~   # or wherever you want
git clone git@github.com:open-horizon/examples.git
```

2. Copy the `processtext` dir to where you will start development of your new service:
```bash
cp -a examples/edge/services/processtext ~/myservice     # or wherever
cd ~/myservice
```

3. Set the values in `horizon/hzn.json` to your own values.

4. Edit `processtext.sh` however you want.
    - Note: This service is a shell script for brevity, but you can write your service in any language.

5. Build the processtext docker image:
```bash
make
```

6. Test the service by having Horizon start it locally:
```bash
hzn dev service start -S
```

7. Check that the container is running:
```bash
sudo docker ps 
```

8. See the processtext service output:
```bash
tail -f /var/log/syslog | grep OVA
```

9. See the environment variables Horizon passes into your service container:
```bash
docker inspect $(docker ps -q --filter name=ibm.processtext) | jq '.[0].Config.Env'
```

10. Stop the service:
```bash
hzn dev service stop
```

11. Have Horizon push your docker image to your registry and publish your service in the Horizon Exchange and see it there:
```bash
hzn exchange service publish -f horizon/service.definition.json
hzn exchange service list
```

12. Publish your edge node deployment pattern in the Horizon Exchange and see it there:
```bash
hzn exchange pattern publish -f horizon/pattern.json
hzn exchange pattern list
```

13. Register your edge node with Horizon to use your deployment pattern (substitute `<service-name>` for the `SERVICE_NAME` you specified in `horizon/hzn.json`):
```bash
hzn register -p pattern-<service-name>-$(hzn architecture) -f horizon/userinput.json
```

14. The edge device will make an agreement with one of the Horizon agreement bots (this typically takes about 15 seconds). Repeatedly query the agreements of this device until the `agreement_finalized_time` and `agreement_execution_start_time` fields are filled in:
```bash
hzn agreement list
```

15. Once the agreement is made, list the docker container edge service that has been started as a result:
```bash
sudo docker ps
```

16. See the processtext service output:

	```
	tail -f /var/log/syslog | grep OVA
	```

17. Unregister your edge node, stopping the processtext service:
```
hzn unregister -f
```

