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

1. nxlog agent should be installed. If not installed, please download the community installer from https://nxlog.co/products/nxlog-community-edition and have it installed in the windows servers.
2. Splunk Universal Forwarder should be installed. If not installed, please download from <a href="https://www.splunk.com/en_us/download/universal-forwarder.html?_gl=1*a91uvm*_ga*NzcxNjkwMjIxLjE2NjUzNjE0NTM.*_ga_5EPM2P39FV*MTY4NzUxNDA0My45MDEuMC4xNjg3NTE0MDQzLjYwLjAuMA..&_ga=2.69836435.261926414.1687152020-771690221.1665361453&_gac=1.48488788.1687187465.CjwKCAjw-b-kBhB-EiwA4fvKrPt1pXmmDzQo_KzDHthk4SBAyzjabM4ygxJ5d1u33REUZSF0wnR-pBoC1UUQAvD_BwE">Splunk Universal Forwarder</a>

