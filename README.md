# splunk-snare-log-format

The purpose of this lab is to test how Splunk read Snare Formatted Windows Event Logs. Conclusion from the test is Splunk does not process Snare logs well and custom parsing will be required to extract the necessary fields. 

## Architecture

Following is a high level setup the the lab environment:

<img width="600" alt="image" src="https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/c1e74ad1-07d6-4e1d-9a8b-971340a1b60a">

There are 3 servers deployed in total: 1 x Windows Server, 1x Splunk Heavy Forwarder, 1 x Splunk Indexer. 

Windows server is installed with both Splunk Universal Forwarder and nxlog agent. nxlog agent collect the windows event logs and output to a file in Snare format. Splunk Universal Forwarder monitors this output file and forward it out for Splunk Heavy Forwarder. 

Splunk Heavy Forwarder receives the logs from Splunk Universal Forwarder and forward it to Splunk Indexer. No indexing is done in this server. 

Splunk Indexer receives the logs from Splunk Heavy Forwarder and index it into the respective indexes. 

Note that all component are installed with Add-on for Windows. 

## Pre-requisites

1. nxlog agent should be installed. If not installed, please download the community installer from <a href="https://nxlog.co/products/nxlog-community-edition">nxlog agent</a> and have it installed in the windows servers.
2. Splunk Universal Forwarder should be installed. If not installed, please download from <a href="https://www.splunk.com/en_us/download/universal-forwarder.html?_gl=1*a91uvm*_ga*NzcxNjkwMjIxLjE2NjUzNjE0NTM.*_ga_5EPM2P39FV*MTY4NzUxNDA0My45MDEuMC4xNjg3NTE0MDQzLjYwLjAuMA..&_ga=2.69836435.261926414.1687152020-771690221.1665361453&_gac=1.48488788.1687187465.CjwKCAjw-b-kBhB-EiwA4fvKrPt1pXmmDzQo_KzDHthk4SBAyzjabM4ygxJ5d1u33REUZSF0wnR-pBoC1UUQAvD_BwE">Splunk Universal Forwarder</a>. Follow this <a href="https://docs.splunk.com/Documentation/Forwarder/9.0.5/Forwarder/Installanixuniversalforwarder">guide</a> to perform the installation
3. Splunk Heavy Forwarder and Splunk Indexer should be installed. Both component use the same installer.

# Part 1 - Setup Splunk Indexer

Step 1: Create indexes to take in the snare logs

Step 2: Ensure receiving port is configured to listen on port 9997. It should be listening by default.

![image](https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/f7acc6ce-32ca-49c7-bd2e-f3a006917856)

# Part 2 - Setup Splunk Heavy Forwarder

Step 1: Configure the forwarding port

<img width="953" alt="image" src="https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/d20a8156-3832-4654-96f9-2c275450c5cc">

Step 2: Ensure receiving port is configured to listen on port 9997. It should be listening by default.

![image](https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/f7acc6ce-32ca-49c7-bd2e-f3a006917856)

# Part 3 - Setup Windows Server

Step 1: Locate the nxlog.conf file in C:\Program Files\nxlog\conf

![image](https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/1dfcb952-e667-42cc-acea-32449c25d4a0)

Step 2: Configure nxlog agent to output window event logs in snare format and save it in a file. Append the following configuration to the last part of nxlog.conf file. 

```
<Input eventlog>
    Module    im_msvistalog
    <QueryXML>
        <QueryList>
            <Query Id='0'>
                <Select Path="System">*</Select>
            </Query>
        </QueryList>
    </QueryXML>
    <Exec>
        # Replace any tabs or newline with a space
        $Message =~ s/(\t|\R)/ /g;

        to_syslog_snare();
    </Exec>
</Input>
<Output out>
    Module      om_file
    File	    "C:\Users\Administrator\Documents\snare\output.log"
</Output>

<Route 1>
    Path        eventlog => out
</Route>
```
Input eventlog instruct nxlog agent to collect Window Event log System. Output out instruct nxlog to output to a file. <Route 1> is to map input and output. 

Step 3: Restart the nxlog agent.

![image](https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/4094c938-30ee-4196-8c42-4294d5532700)

Step 4: Verify that the Windows event log is saved in the output directory. 

![image](https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/3467fa86-722d-4107-a15a-30e0818bbcf1)

Step 5: Configure Splunk Universal Forwarder to monitor the snare output file. Locate the inputs.conf in C:\Program Files\SplunkUniversalForwarder\etc\system\local. If the local folder or the inputs.conf is not  found, feel free to create it yourself. 

![image](https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/677a91c4-735c-401f-8125-16cf04832955)

Step 6: Paste the following configuration into the inputs.conf file

```
[default]
host=splunkwindows1

[monitor://C:\Users\Administrator\Documents\snare]
sourcetype = windows_snare_syslog
index = <your index>
```

Step 7: Configure Splunk Universal Forwarder to forward the logs out to Splunk Heavy Forwarder. In the same folder (C:\Program Files\SplunkUniversalForwarder\etc\system\local), locate outputs.conf

![image](https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/d5785c24-9373-4931-ae17-a2c3182aacf5)

Step 8: Paste the following configuration into the outputs.conf file

```
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <your heavy forwarder IP>:9997

[tcpout-server://<your heavy forwarder IP>:9997]
```

Step 8: Restart Splunk Universal Forwarder Service

![image](https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/86a21c3e-5fce-4c37-8408-9eecd858a8bc)



