---
title: Contact
nav:
  order: 5
  tooltip: Email, address, and location
---

# {% include icon.html icon="fa-regular fa-envelope" %}Contact

The Cornett Lab is housed within the Indiana University School of Medicine Department of Biochemistry and Molecular Biology. The lab is located within the Van Nuys Medical Science building on the Indiana University - Indianapolis campus.<br>

**Shipping Address:**<br>
ATTN: Cornett Lab<br>
Building VanNuys Med Sci Bldg<br>
Room# MS4075<br>
635 BARNHILL DR<br>
INDIANAPOLIS, IN 46202-5120<br>
United States<br>

{%
  include button.html
  type="email"
  tooltip="Email Evan"
  text="evcorn@iu.edu"
  link="evcorn@iu.edu"
%}
{%
  include button.html
  type="phone"
  tooltip="Evan's Office Phone"
  text="(317) 278-4503"
  link="+1-317-278-4503"
%}
{%
  include button.html
  type="address"
  text="Map"
  tooltip="Our location on Google Maps for easy navigation"
  link="https://www.google.com/maps?sll=39.776612,-86.178275&q=635+Barnhill+Drive+Indianapolis,+IN,+46202,+United+States&z=16"
%}

{% include section.html %}

{% capture col1 %}

{%
  include figure.html
  image="images/indianapolis.jpg"
  
%}

{% endcapture %}

{% capture col2 %}

{%
  include figure.html
  image="images/DNA.jpg"
  
%}

{% endcapture %}

{% include cols.html col1=col1 col2=col2 %}



{% include section.html dark=true %}

{% capture col1 %}

{% endcapture %}

{% capture col2 %}

{% endcapture %}

{% capture col3 %}

{% endcapture %}

{% include cols.html col1=col1 col2=col2 col3=col3 %}
