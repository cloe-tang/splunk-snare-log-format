# splunk-snare-log-format

The purpose of this lab is to test how Splunk read Snare Formatted Windows Event Logs. Conclusion from the test is Splunk does not process Snare logs well and custom parsing will be required to extract the necessary fields. 

Following is a high level setup the the lab environment:

<img width="600" alt="image" src="https://github.com/cloe-tang/splunk-snare-log-format/assets/58005106/c1e74ad1-07d6-4e1d-9a8b-971340a1b60a">

There are 3 servers deployed in total: 1 x Windows Server, 1x Splunk Heavy Forwarder, 1 x Splunk Indexer. 

Windows server is installed with both Splunk Universal Forwarder and nxlog agent. nxlog agent collect the windows event logs and output to a file in Snare format. Splunk Universal Forwarder monitors this output file and forward it out for Splunk Heavy Forwarder. 

