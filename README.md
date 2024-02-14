# Synology Overly Comprehensive Dashboard (OCD)

A complete Docker stack to run a Grafana Dashboard on a Synology NAS.

Grafana | Prometheus | cAdvisor | Speedtest Exporter | Node Exporter | SNMP Exporter | UPS stats

## Setup

For each NAS you want to collect the data from, follow the steps below.

Before you start, make sure you have the Container Manager package installed via the Synology DSM.

### Firewall

Check if your Firewall is enabled: **Control Panel** -> **Security** -> **Firewall**

If the **Enable firewall** checkbox is not ticked, move to the next section. 

If the checkbox *is* ticked, do the following:

1. Under **Firewall Profile** click the **Edit Rules** button

2. Under **Firewll Rules** click the **Create** button

3. Set **Ports** to **All**

4. Under **Source IP** select Specific IP and click the **Select** button

5. Choose the Subnet option and enter IP address `172.22.0.0` and Subnet mask `255.255.0.0` and press **OK**

6. Set the **Action** to **Allow**, make sure the Enabled checkbox is ticked, and click **OK**

7. Make sure this newly created rule is on top of all the other Firewall Rules. Drag and drop it to the top.

### SNMP Exporter

1. Open **Control Panel** -> **Terminal and SNMP** -> **SNMP** and tick the **Enable SNMP service** checkbox.

2. Tick the **SNMPv3 service** checkbox. Set the username to `exporter`, protocol to `MD5`, and pick a password.

3. Tick the **Enable SNMP privacy** checkbox. Set protocol to DES and pick a second, different password.

4. Edit the `snmp.yml` file and replace the `snmpv3_password_placeholder` and `snmp_priv_password_placeholder` with the passwords you just picked.

### Prometheus

Replace the `your_real_host_ip` placeholder with your own local NAS IP (for example 192.168.0.10).

### Docker compose

Now, pick a user you'd like to dedicate for running Docker on your NAS. It can be an existing user, or you can create a brand new user just for that.

You will need to enable SSH and then SSH into your system and check the user's gid and uid. You can do that by running the command `id username`. For example, if I have a user called `dockeruser`, I will run `id dockeruser` and see the output `uid=1028(dockeruser) gid=100(users) groups=100(users)`.

Depending on whether you're deploying on your main NAS that will also host the Grafana frontend (master), or a second or third device you want to add to the pool (slave), pick the right `compose.yaml` file. Open the file in an editor and replace all instances of the `user_id:group_id` placeholder with the uid and gid, for example `1028:100`.

### File structure

In this step, you will need to make sure all the necessary directories and files are in place.

1. Using the DSM File Station, navigate to the docker shared folder. Inside the docker share, create an empty directory called `metrics`. Inside the `metrics` directory you just created, create three more empty directories: `grafana`, `prometheus`, and `snmp`.

2. Upload your modified `prometheus.yml` and `snmp.yml` files into the `metrics` directory

3. Your final file structure should look something like this:

```
volume1
└── docker
    └── metrics
        ├── prometheus.yml
        ├── snmp.yml
        ├── prometheus
        ├── grafana
        └── snmp
```

### Docker setup

1. Open **Container Manager** -> **Project** and click **Create**

2. Enter the **Project name** `metrics`

3. Set the **Path** to `/docker/metrics`

4. Upload the `master/compose.yaml` or `slave/compose.yaml` file depending on whether it is your main or secondary NAS.

5. Deploy the project!

### Grafana setup

1. Go to http://your-nas-ip:3340/ (replace `your-nas-ip` with your actual NAS IP address or domain name)

2. Log in to Grafana with the default user `admin` and password `admin`

3. You will be prompted to select a new password. Make sure you write it down somewhere!

4. Go to **Home** -> **Connections** -> **Add new connection**

5. Search for **Prometheus**

6. Click **Add new data source** on the top right

7. Choose and enter a name for the NAS (e.g. `nas1` or `nas2`) and enter the **Prometheus server URL**:
- If this is your master NAS (running Grafana), enter `http://prometheus:9090`
- If this is your slave NAS (no Grafana, only Prometheus), enter `http://your-nas-ip:9090/` but make sure to replace `your-nas-ip` with the IP address of the remote (slave) NAS

8. Leave everything else as default and click **Save & test**

### Import the Dashboard

This is an additional step you only need to follow once on your master NAS.

1. Open Grafana (http://your-nas-ip:3340/) and on the left side menu select Dashboards

2. On the top right click **New** -> **Import**

3. Click **Upload dashboard JSON file** and upload the `Synology_Dashboard.json` file

4. Click **Import**. You should see your Dashboard now!
