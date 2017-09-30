
This Vagrantfile will spawn 4 instances of VQFX (Full) each with 1 Routing Engine and 1 PFE VM  
All VQFX will be interconnected in a full-mesh

U used vqfx to deploy all the stuff. Ansible workbooks are work in progress.

I made connections using port-map dictionary which i used in Vagrantfile.
I'm not sure whether it scales for a bit topology , but it works just fine and very easy to use


Below is actual scheme

![alt text](https://www.lucidchart.com/publicSegments/view/4220650f-d696-424b-bbe0-9db2ef3cdf88/image.png)
