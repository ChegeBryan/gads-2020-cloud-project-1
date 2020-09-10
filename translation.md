# LAB - Google Cloud Fundamentals: Getting Started with Compute Engine

## Objectives

- Create a Compute Engine virtual machine using the gcloud command-line interface.

- Connect between the two instances.

1. Setup your cloud shell environment
   Open your Cloud shell terminal. Once the initialization is complete we will be ready to start.

   - Config the project to use for this session, (assume project name is, `sky-high-cloud-12345`)

   ```
   export PROJECT_ID=sky-high-cloud-12345
   ```

   - Next, tell `gcloud` about our project,

   ```
   gcloud config set project $PROJECT_ID
   ```

   - If you are prompted to authorize, allow that option.
   - Setting up the default resources zone. To check what zones and regions are available, run:

   ```
   gcloud compute zones list
   ```

   The command lists the available zones and what regions are UP. For this lab we will use `us-central1-a` zone.

   - Lets tell our terminal about it, run:

   ```
   export my_zone1=us-central1-a
   ```

2. Create our first VM instance
   We are going to create a vm instance in our `us-central1-a` zone and will use a `debian-9-stretch` image, run:

   ```
   gcloud compute instances create first-vm \
   --zone $my_zone1 \
   --machine-type n1-standard-1 \
   --image-project debian-cloud \
   --image debian-9-stretch-v20200902 \
   --subnet default
   ```

   - Our VM name is `first-vm`
   - The `--machine-type` specifies that we want to use a `n1-standard-1` machine which gives us a `1 vCPU, 3.75 GB memory` configuration for our VM.
   - The `--image-project` specifies the Google Cloud project against which all image and image family references will be resolved. For our instance we are going to use `debian-cloud` project and our `--image` of choice is `debian-9-stretch-v20200902`
   - The `--subnet` flag specifies the subnet the vm instance will be part of. For our instance we set it to use the `default` subnet interface. By default new projects start with a default network (an auto mode VPC network) that has one subnetwork (subnet) in each region, the subnet name is `default`.

3. Edit firewall rules
   We are going to `allow HTTP` traffic to our VM instance, run:

   ```
   gcloud compute firewall-rules create default-allow-http \
   --direction=INGRESS \
   --network=default \
   --action=ALLOW \
   --rules=tcp:80
   ```

   - We are creating a firewall rule and named it `default-allow-http`.
   - We have specified the direction using the `--direction` flag. The set value is `INGRESS` (could have also typed `IN`), This allows ingress traffic to our instance.
   - `--network` we are attaching to rule our `default` network.
   - `--action` flag is set to `ALLOW` which affects how `--rules` value is treated. For this instance we are allowing `tcp` protocol connections over port `80`.

4. Create our second VM instance
   To create our second Vm instance, run:

   ```
   gcloud compute instances create second-vm \
   --machine-type n1-standard-1 \
   --zone us-central1-b \
   --image-project debian-cloud \
   --image debian-9-stretch-v20200902 \
   --subnet default
   ```

   - Note for the second we have set the zone to be at `us-central1-b`.

5. SSH to `first-vm`
   To connect to our `first-vm` ssh instance, run:

   ```
   gcloud compute ssh my-vm-2 --zone us-central1-a
   ```

   - When prompted whether to continue connecting to a host with unknown authenticity, enter `yes` to confirm that you do.
   - Once the connection is initiated a prompt for generating public/private rsa key pair will ask for a passphrase. Leave empty and let the process continue.
   - Once finished the command prompt will switch to the our VM instance.

6. Installing nginx web server on `first-vm`
   On the command prompt for `first-vm`, run:

   ```
   sudo apt update
   ```

   - The command will update our our vm package repository on our vm.
   - Next to install nginx-light web server, run:

   ```
   sudo apt install nginx-light -y
   ```

   - Use the nano text editor to add a custom message to the home page of the web server, run:

   ```
   sudo nano /var/www/html/index.nginx-debian.html
   ```

   - Inside the file add the following text just below the `h1`:

   ```
   Hi added text.
   ```

   - Press `Ctrl+O` and then press `Enter` to save your edited file, and then press `Ctrl+X` to exit the nano text editor.
   - Confirm that the web server is serving your new page. At the command prompt on `first-vm`, run:

   ```
   curl localhost
   ```

7. Confirming we can access `first-vm` from our `second-vm`
   On the command line type in `exit`. You will be taken back to your cloud shell.

   - On the command prompt switch to the `second-vm` instance, run:

   ```
   gcloud compute ssh second-vm --zone us-central1-b
   ```

   - To confirm we can access `first-vm` from `second-vm`, run:

   ```
   ping first-vm
   ```

   - To confirm we can get back the html page from `first-vm` on `second-vm`, run:

   ```
   curl http://first-vm/
   ```

   - You should get back the html page that we had edited earlier.
