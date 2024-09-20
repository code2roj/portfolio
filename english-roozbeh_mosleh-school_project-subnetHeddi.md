# subnetHeddi: IP Address and Subnet Calculator

  
## Overview

  

This Python program was written during my preparatory course at BITLC Dortmund. While attending a network module, I developed this project to gain a hands-on understanding of networking concepts. Built using the Tkinter GUI library, this tool helps calculate key network information such as the subnet mask, network address, broadcast address, host range, and the number of available IP addresses. The user inputs an IP address and a CIDR prefix, and the program displays the results in both binary and decimal formats.

![Image 1](attachments/img_01-subnetHeddi-gui-screenshot.jpeg)

---

## Features

  

- **IP Address Input:** Enter the IP address in the standard IPv4 format (e.g., `192.168.1.1`).

- **CIDR Input:** Enter the subnet mask in CIDR notation (e.g., `24` for a `/24` subnet).

- **Calculations:**

- Binary and Decimal Subnet Mask

- Network Address (Binary and Decimal)

- Broadcast Address (Binary and Decimal)

- Host Min and Max

- Available IP Addresses

- **Results Displayed in the Middle of the Window:** Results are shown on top of the background image, centered above the input fields.

  

## Requirements

  

- Python 3.x

- Tkinter (Comes with most Python distributions)

- PIL (Pillow library for image handling)

  

To install Pillow, use the following command:

  

```bash

pip install pillow

```

  

## How to Use

  

1. Run the Python program.

2. Enter an IP address in the input field for "Enter the IP address".

3. Enter the CIDR prefix in the input field for "Enter the Prefix (CIDR)".

4. Click the "Compute" button.

5. The result will be displayed in the middle of the window.

  

## Example

  

- **Input:**

IP Address: `192.168.1.1`

CIDR Prefix: `24`

  

- **Output:**

```

IP Address: 192.168.1.1

IP Address (Binary): 11000000.10101000.00000001.00000001

Subnet Mask: 255.255.255.0

Subnet Mask (Binary): 11111111.11111111.11111111.00000000

Network Address: 192.168.1.0

Network Address (Binary): 11000000.10101000.00000001.00000000

Broadcast Address: 192.168.1.255

Broadcast Address (Binary): 11000000.10101000.00000001.11111111

Host Min: 192.168.1.1

Host Max: 192.168.1.254

Available IPs: 254

```

  

## Source Code

```python
import tkinter as tk

from tkinter import Label, Frame, simpledialog

from PIL import Image, ImageTk

import os

  

# Haupt-Fenster erstellen

haupt_fenster = tk.Tk()

haupt_fenster.title("BITLC-Schulprojekt von Roozbeh Mosleh")

haupt_fenster.maxsize(3000, 2500) # Breite x Höhe

haupt_fenster.minsize(900, 750) # Breite x Höhe

haupt_fenster.config(bg="#006666")

  

# Load and display banner image

banner_path = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'bg.png')

banner_img = Image.open(banner_path)

# Resize the image if needed

banner_resized = banner_img.resize((900, 750), Image.LANCZOS)

banner_photo = ImageTk.PhotoImage(banner_resized)

  

banner_label = Label(haupt_fenster, image=banner_photo, bg="white", relief=tk.SUNKEN)

banner_label.image = banner_photo # Keep a reference

banner_label.pack()

  

# Create the label to display the result, place it in the middle of the window, above the input fields

result_label = tk.Label(haupt_fenster, text="", font=("Helvetica", 13, "bold"), fg="white", bg="#006666", justify="left")

result_label.place(relx=0.5, rely=0.4, anchor='center')

  

# Create a frame for inputs

input_frame = tk.Frame(haupt_fenster, bg="#006666")

input_frame.place(relx=0.5, rely=0.6, anchor='n') # Place below the result_label

  

# IP Address label and entry

ip_label = tk.Label(input_frame, text="Enter the IP address:", font=("Helvetica", 13), fg="white", bg="#006666")

ip_label.grid(row=0, column=0, padx=5, pady=5)

ip_entry = tk.Entry(input_frame, font=("Helvetica", 13))

ip_entry.grid(row=0, column=1, padx=5, pady=5)

  

# Prefix label and entry

prefix_label = tk.Label(input_frame, text="Enter the Prefix (CIDR):", font=("Helvetica", 13), fg="white", bg="#006666")

prefix_label.grid(row=1, column=0, padx=5, pady=5)

prefix_entry = tk.Entry(input_frame, font=("Helvetica", 13))

prefix_entry.grid(row=1, column=1, padx=5, pady=5)

  

# Compute button

compute_button = tk.Button(haupt_fenster, text="Compute", font=("Helvetica", 13), command=lambda: compute_results())

compute_button.place(relx=0.5, rely=0.75, anchor='n')

  

def compute_results():

ip_address = ip_entry.get()

cidr_prefix = prefix_entry.get()

try:

sliced_ip_list = ip_address.split(".")

if len(sliced_ip_list) != 4:

raise ValueError("Invalid IP address format")

ip_octets_int = []

bin_ip_list = []

for element in sliced_ip_list:

int_element = int(element)

if not 0 <= int_element <= 255:

raise ValueError("IP address octets must be between 0 and 255")

ip_octets_int.append(int_element)

bin_ip_list.append(format(int_element, '08b'))

cidr_prefix = int(cidr_prefix)

if not 0 <= cidr_prefix <= 32:

raise ValueError("CIDR prefix must be between 0 and 32")

eins = cidr_prefix * '1'

nulls_count = 32 - cidr_prefix

nulls = nulls_count * '0'

subnet_eins_nulls = eins + nulls

# Divide binary subnet into 4 slices

s1 = subnet_eins_nulls[:8]

s2 = subnet_eins_nulls[8:16]

s3 = subnet_eins_nulls[16:24]

s4 = subnet_eins_nulls[24:32]

bin_subnet_list = [s1, s2, s3, s4]

dec_subnet_list = [int(element, 2) for element in bin_subnet_list]

# Network address

dec_network_address_ls = [ip & subnet for ip, subnet in zip(ip_octets_int, dec_subnet_list)]

bin_network_address_ls = [format(addr, '08b') for addr in dec_network_address_ls]

# Broadcast address

dec_wildcard_mask_list = [255 - subnet for subnet in dec_subnet_list]

dec_broadcast_address_ls = [network | wildcard for network, wildcard in zip(dec_network_address_ls, dec_wildcard_mask_list)]

bin_broadcast_address_ls = [format(addr, '08b') for addr in dec_broadcast_address_ls]

# Host min and max

host_min = dec_network_address_ls[:]

host_min[-1] += 1

host_max = dec_broadcast_address_ls[:]

host_max[-1] -= 1

# Number of available IPs

available_ips = (2 ** (32 - cidr_prefix)) - 2 # Exclude network and broadcast addresses

# Prepare data for display

display_result_show = display_result(sliced_ip_list, bin_ip_list, dec_subnet_list, bin_subnet_list,

dec_network_address_ls, bin_network_address_ls,

dec_broadcast_address_ls, bin_broadcast_address_ls,

available_ips, host_min, host_max)

# Update result_label

result_label.config(text=display_result_show)

except ValueError as e:

result_label.config(text=str(e))

  

def display_result(sliced_ip_list, bin_ip_list, dec_subnet_list, bin_subnet_list,

dec_network_address_ls, bin_network_address_ls,

dec_broadcast_address_ls, bin_broadcast_address_ls,

available_ips, host_min, host_max):

result_text = f"IP Address: {'.'.join(sliced_ip_list)}\n" \

f"IP Address (Binary): {'.'.join(bin_ip_list)}\n" \

f"Subnet Mask: {'.'.join(map(str, dec_subnet_list))}\n" \

f"Subnet Mask (Binary): {'.'.join(bin_subnet_list)}\n" \

f"Network Address: {'.'.join(map(str, dec_network_address_ls))}\n" \

f"Network Address (Binary): {'.'.join(bin_network_address_ls)}\n" \

f"Broadcast Address: {'.'.join(map(str, dec_broadcast_address_ls))}\n" \

f"Broadcast Address (Binary): {'.'.join(bin_broadcast_address_ls)}\n" \

f"Host Min: {'.'.join(map(str, host_min))}\n" \

f"Host Max: {'.'.join(map(str, host_max))}\n" \

f"Available IPs: {available_ips}"

return result_text

  

haupt_fenster.mainloop()
```
