# SSTI-Basics
Basic overview of SSTI to RCE

Server-side template injection is when an attacker is able to use native template syntax to inject a malicious payload into a template, which is then executed server-side.

Template engines are designed to generate web pages by combining fixed templates with volatile data. Server-side template injection attacks can occur when user input is concatenated directly into a template, rather than passed in as data. This allows attackers to inject arbitrary template directives in order to manipulate the template engine, often enabling them to take complete control of the server. As the name suggests, server-side template injection payloads are delivered and evaluated server-side, potentially making them much more dangerous than a typical client-side template injection. 

```
sudo apt-get install toilet && apt-get install figlet
git clone https://github.com/RoqueNight/HydraSpliot.git
cd HydraSpliot
chmod +x hydraspliot
./hydraspliot
```
