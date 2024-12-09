[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-24ddc0f5d75046c5622901739e7c5dd533143b0c8e959d652212380cedb1ea36.svg)](https://classroom.github.com/a/kzkUPShx)
# a14g-final-submission

    * Team Number: 5
    * Team Name: Pruduct Scanner
    * Team Members: Tianshu Wang & Jin Qian
    * Github Repository URL: https://github.com/ese5160/a14g-final-submission-t05-product-scanner
    * Github Pages URL: https://ese5160.github.io/a14g-final-submission-t05-product-scanner/
    * Description of test hardware: 
         SAMW25 Microcontroller
         SHTC3-TR-10KS Sensor
         1.8" Color TFT LCD with SPI Interface
         Tiny Code Reader from Useful Sensors
         ADA1201 Vibrating Motor
         3.7V Li-Ion Battery

## 1. Video Presentation
<div align="center">
  <a href="https://www.youtube.com/watch?v=oSgTU4lZSk4">
    <img src="https://img.youtube.com/vi/oSgTU4lZSk4/0.jpg" alt="Video Thumbnail" width="500">
  </a>
</div>


## 2. Project Summary
### Device Description
Our device serves as a supermarket product scanner, allowing customers to scan QR codes to access item information such as location, name, and stock quantity displayed on an LCD screen. Supermarket staff can monitor the products scanned by customers and their respective locations via the terminal web page. Moreover, employees can also track the temperature and humidity within the supermarket to assess if environmental adjustments are necessary.

### Inspiration
The inspiration for this product mainly came from what we saw in supermarkets with large indoor areas (such as Costco and Walmart). When we shop in these supermarkets, we often encounter the situation where we want to buy a small product, but it is difficult to find the product because the supermarket is too large. Therefore, our team wants to design a device that can scan the product's QR code. After scanning the QR code, the specific location of the item you are looking for, as well as the price and remaining quantity information can be displayed. Additionally, this product has a built-in temperature and humidity sensor. This information will be sent back to the computers of supermarket staff, allowing them to monitor the environment in the supermarket and adjust the air conditioning temperature in time to ensure that the products are fresh enough to prevent deterioration. Therefore, the target customers of this product are the operators of these large supermarkets. It can be installed on supermarket shopping carts for customers to use.

### Device Functionality
We have designed a device capable of scanning QR codes to retrieve product information while simultaneously monitoring indoor temperature and humidity. The data collected by this device can be transmitted over the network and displayed on the terminal web page using Node-RED. Our device consists of two sensors and two actuators. The sensors include an SHTC3 temperature and humidity sensor and a QR code scanner, while the actuators comprise a vibrating motor and a color LCD screen.

The overall workflow of our product is as follows:
1. When a customer uses the device to scan a QR code, the QR code scanner identifies the scanned QR code and provides vibration feedback to the customer using the vibrating motor.
2. The information retrieved from the scan is displayed on the LCD screen, showing the product's location, name, and inventory quantity.
3. The location information of the scanned items is also uploaded to the Node-RED UI interface, allowing access to this information via devices such as computers or smartphones.
4. Finally, the SHTC3 sensor installed within our device continuously monitors the temperature and humidity of the supermarket. This information is uploaded to the Node-RED UI interface for display. Additionally, if the temperature exceeds a certain threshold or the humidity reaches a high level, a pop-up warning will appear on the page, alerting staff to adjust the air conditioning settings in that area.

### Challenges
In general, we faced four challenges and partly solved them in the right way.

#### 1. LCD Configuration
Our initial challenge stemmed from the SPI configuration for the LCD screen. Upon configuring the LCD, we encountered difficulty as the designated pin ports of the MCU couldn't be configured with SPI. This oversight likely occurred due to not verifying the configuration in Atmel Start during PCB design. Instead, we relied on the SPI line indicated on the MCU in the Altium schematic.
Our resolution involved selecting alternative pins since the connector allocated for the LCD had eight pins, yet only six were utilized. Thus, we utilized the remaining two pins. Additionally, configuring the SPI pin port proved challenging as we deliberated over which SPI module and SERCOM to choose. Eventually, we consulted Microchip Studio's official website documentation and the SAMW25 datasheet to determine the appropriate configuration for the SPI module.

#### 2. PCB Design Issues
Our second challenge revolved around various schematic and PCB design issues encountered with our board. One such instance arose when designing the circuit to drive a vibrating motor. Initially, we replicated the typical design outlined in the motor's datasheet. However, upon testing, we discovered that this setup resulted in both ends of the motor being high when one should have been high and the other low. Ultimately, we rectified this by implementing the correct circuit externally to the PCB board.
Another issue arose with certain pin ports that were unable to be driven, such as pulling them low or high. Despite theoretically being able to utilize all pins as GPIO, we encountered discrepancies where some pins responded as expected while others did not. We speculated that certain pin lines might be too lengthy, causing disconnections midway. Our workaround involved utilizing only those pins that functioned properly.

#### 3. FreeRTOS Heap Task Size
Our third challenge involved managing tasks with varying sizes and priorities, leading to mutual interference. Initially, when operating with only two tasks, they generally cooperated correctly but occasionally encountered issues like getting stuck. However, upon introducing a third task, inter-task interference became frequent, causing program malfunctions.
To address this, we merged the two tasks into a single one, which effectively eliminated the encountered problems. Additionally, we implemented the high-water mark function to accurately measure the size of each task, ensuring their proper functioning.

#### 4. QR Code Scanner Buffer Data Transmission - Node-RED + LCD Screen
Our fourth challenge revolved around the data transmission issue of the QR code reader. We aimed to retrieve comprehensive product information, necessitating the output of a string of characters. However, we encountered difficulty in correctly writing the buffer to transmit this string, which consumed a significant amount of time in coding efforts. Eventually, through persistent troubleshooting, we successfully displayed the accurate product information on the LCD screen and the product location on the Node-RED website.

### Prototype Learnings
Reflecting on the process of building and testing our prototype, where we created and utilized our own PCB, several critical lessons emerged. First, the utility of test points was evident; they greatly facilitated troubleshooting and functional verification at various stages. When selecting pins, it is essential to ensure they align with the intended use and hardware compatibility, which is the most significant part of implementing PCB in the real project.

Another important realization was the necessity of depopulating jumpers in the manufacturing stage to easily test the functionality of different modules. This practice helps in isolating problems and making incremental adjustments without impacting the entire system.

The power system’s integrity proved to be crucial. Modifying standard components can lead to unexpected failures and malfunctions because the components are standard designed and cannot change the pin location.

Lastly, while datasheets are invaluable resources, maintaining a healthy skepticism about the circuit configurations they suggest is beneficial. It's vital to validate each suggested configuration against practical application scenarios to ensure they meet the specific requirements of our project. If tasked with building this device again, these insights would guide us to optimize the design and testing phases to achieve a more robust and reliable outcome.

### Next Steps
In terms of hardware, our next step is to re-modify our own PCBA board and solve the previous wiring and connector problems so that the board will no longer have any physical errors and will be more convenient for subsequent use. At the same time, a module for communicating with the Internet is installed on the PCBA to improve the functionality of this product.

At the software level, you can add a database stored in the cloud to save the customer's username and password, so that my node-red customer login interface can truly become a function that can be used to log in to secondary pages. At the same time, the information inside the QR code is also uploaded to the cloud. When the product is sold, the QR code information can be synchronized to update the remaining quantity of the product.

### Takeaways from ESE5160
What did you learn in ESE5160 through the lectures, assignments, and this course-long prototyping project?
1. The first and most important thing is to learn how to design PCBA boards through Altium, and know how to place components and correctly connect circuits.
2. Learned how to test and debug hardware through oscilloscopes and other instruments, find circuit faults and find ways to solve them.
3. Learned how to design a UI interface for your own product through node-red. Through node-red, I can try to think from the customer's perspective on how to design the UI interface to be more concise and easy to use.
4. Learn how to write code correctly to ensure that various components have enough heap space to allow them to run in an orderly and continuous manner.

### Project Codebase Links
- **Node-RED instance:** [http://52.177.130.52:1880/ui](http://52.177.130.52:1880/ui)
- **A12G Code Repository:** [GitHub Repository](https://github.com/ese5160/a12g-firmware-drivers-t05-product-scanner)
- **Final PCBA on Altium 365:** [Altium 365 Link](https://upenn-eselabs.365.altium.com/designs/CA7AC7AB-A449-4488-9C62-C8DDCABF01EC)

## 3. Hardware & Software Requirements

### Hardware Requirements

#### HRS 01 - SAMW25 Microcontroller
Our project is based on a self-designed PCB. The MCU of this PCB is the SAMW25 microcontroller, with J-link used to flash the code into the PCB.

#### HRS 02 - SHTC3-TR-10KS Sensor
In compliance with HRS 02, our system employs the SHTC3-TR-10KS sensor, which detects a wide range of temperature (-40 °C to 125 °C) and humidity (0 to 100 %RH). The typical accuracy is ±2 %RH for humidity and ±0.2°C for temperature, but our software configuration displays accuracy to ±1°C for temperature and ±1 %RH for humidity due to integer data type limitations.

#### HRS 03 - 1.8" Color TFT LCD with SPI Interface
The implementation of the 1.8" Color TFT LCD with an SPI interface in our project was crucial for displaying detailed product information. This LCD module communicates with the microcontroller through the SPI, ensuring swift and stable data updates.

#### HRS 04 - Tiny Code Reader from Useful Sensors
The Tiny Code Reader captures and interprets QR codes effectively, with an LED indicator that signals the scanning process status. It communicates with the microcontroller via I2C, changing from blue to green upon a successful scan.

#### HRS 05 - ADA1201 Vibrating Motor
The ADA1201 vibrating motor is integrated as a tactile feedback mechanism. It sends a vibration alert each time an item is scanned and when the user finishes shopping, enhancing the user experience with immediate physical feedback.

#### HRS 06 - System Integration and Control
Our development team has implemented a system that integrates multiple hardware components to ensure smooth and efficient operation. The device operates on a 3.7V Li-Ion battery, with a tested continuous operational capability of at least 3 hours.

### Software Requirements

#### SRS 01 - QR Code Scanning and Processing
The Adafruit 5744 QR code reader scans QR codes on products with a response time of less than 2 seconds per scan. While the I2C protocol speed is assumed to be efficient, we lack tools to measure the exact speed, focusing on the operational performance.

#### SRS 02 - Display Management
We manage a 1.8" Color TFT LCD screen with an SPI interface, capable of displaying product details. Current performance shows a refresh time of approximately one to two seconds, slower than the desired 500 milliseconds.

#### SRS 03 - Temperature and Humidity Monitoring
The software interfaces with the SHTC3-TR-10KS sensor, collecting and transmitting temperature and humidity data every 5 seconds via MQTT to Node-RED for real-time monitoring and analysis.

#### SRS 04 - Wireless Data Transmission
Using the SAMW25 Wi-Fi microcontroller, our system handles data transmission over a 2.4 GHz Wi-Fi network, supporting 802.11 b/g/n standards with a minimum data transfer rate of 150 Mbps. It seamlessly connects to various networks and interfaces with different server configurations.

#### SRS 05 - User Interaction and Feedback
The ADA1201 vibrating motor is controlled by software to activate for 0.5 seconds for scan confirmations and alerts, with a slight delay in the vibration timing due to code issues but still delivering timely feedback.


## 4. Project Photos & Screenshots

### Final Project Overview
<p align="center">
  <img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/project%20whole%20view.jpg" alt="Project Whole View">
</p>
*Description: Image of the complete final project setup, showcasing all external components like 3D prints, screens, and buttons.*

### Standalone PCBA - Top View
*Description: Top view of the standalone printed circuit board assembly (PCBA).*
<p align="center">
  <img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/top%20view.jpg" alt="Top View of the Project">
</p>

### Standalone PCBA - Bottom View
<p align="center">
  <img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/bottom%20view.jpg" alt="Bottom View of the Project">
</p>
*Description: Bottom view of the standalone printed circuit board assembly (PCBA).*

### Thermal Camera Images
<p align="center">
  <img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/heat.jpg" alt="Thermal Image of the Project">
</p>
*Description: Thermal camera image showing the board while running under load. Useful for assessing thermal performance and hot spots.*

### Altium Board Design - 2D View
<p align="center">
  <img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/Altium2D.png" alt="Altium 2D Design View">
</p>
*Description: Screenshot of the 2D view of the Altium board design, highlighting the layout and component placement.*

### Altium Board Design - 3D View
<p align="center">
  <img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/Altium3D.png" alt="Altium 3D Design View">
</p>
*Description: Screenshot of the 3D view of the Altium board design, providing a realistic visualization of the assembled board.*

### Node-RED Dashboard
<div align="center">
<table><tr>
<td><img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/Login%20page.png" width="1000"/></td>
<td><img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/Data%20page.png" width="1000"/></td>
</tr><tr>
<td><img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/OTA%20reset.png" width="1000"/></td>
<td><img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/Qr%20massage.png" width="1000"/></td>
</tr></table>
</div>
*Description: Screenshot of the Node-RED dashboard, displaying the user interface and data visualization components.*

### Node-RED Backend
<div align="center">
<table><tr>
<td><img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/Userpage.png" width="1000"/></td>
<td><img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/Qrcode%20scanner.png" width="1000"/></td>
</tr><tr>
<td colspan="2"><img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/OTAFU.jpg" width="1000"/></td>
</tr></table>
</div>
*Description: Screenshot of the Node-RED backend, illustrating the flow and logic used to process data.*

### System Block Diagram
<p align="center">
  <img src="https://github.com/ese5160/a14g-final-submission-t05-product-scanner/blob/main/Screenshoot%20and%20photo/System_diagram.png" alt="System Block Diagram">
</p>
*Description: Updated block diagram of the system, reflecting changes and integration details from the semester.*

## 5. Project Codebase Links
- **Node-RED instance:** [http://52.177.130.52:1880/ui](http://52.177.130.52:1880/ui)
- **A12G Code Repository:** [GitHub Repository](https://github.com/ese5160/a12g-firmware-drivers-t05-product-scanner)
- **Final PCBA on Altium 365:** [Altium 365 Link](https://upenn-eselabs.365.altium.com/designs/CA7AC7AB-A449-4488-9C62-C8DDCABF01EC)
