#Windows 

# Alternate Data Streams

[About alternate data streams](https://www.malwarebytes.com/blog/news/2015/07/introduction-to-alternate-data-streams) by Malwarebytes:
- Alternate Data Streams (ADS) are a file attribute only found on the [NTFS file system](https://technet.microsoft.com/en-us/library/cc781134(v=ws.10).aspx).
- In this system a file is built up from a couple of attributes, one of them is _$Data_, aka the data attribute. Looking at the regular data stream of a text file there is no mystery. It simply contains the text inside the text file. But that is only the primary data stream.
- This one is sometimes referred to as the unnamed data stream since the name string of this attribute is empty. So any data stream that has a name is considered alternate.
- These data streams suffer from a bad reputation since they have been used and abused to write hidden data. Varying from data about where a file came from to complete malware files (e.g. [Backdoor.Rustock.A](https://www.symantec.com/security_response/writeup.jsp?docid=2006-060111-5747-99&tabid=2))

