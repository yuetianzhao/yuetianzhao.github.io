[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/4PO5M1Wx)
# final-project-skeleton

    * Team Name: 
    * Team Members: 
    * Github Repository URL: 
    * Github Pages Website URL: [for final submission]
    * Description of hardware: (embedded hardware, laptop, etc) 

## Final Project Report

Don't forget to make the GitHub pages public website!
If you’ve never made a Github pages website before, you can follow this webpage (though, substitute your final project repository for the Github username one in the quickstart guide):  <https://docs.github.com/en/pages/quickstart>

### 1. Video

[Insert final project video here]

### 2. Images

[Insert final project images here]

### 3. Results

What were your results? Namely, what was the final solution/design to your problem?

#### 3.1 Software Requirements Specification (SRS) Results

Based on your quantified system performance, comment on how you achieved or fell short of your expected software requirements. You should be quantifying this, using measurement tools to collect data.

#### 3.2 Hardware Requirements Specification (HRS) Results

Based on your quantified system performance, comment on how you achieved or fell short of your expected hardware requirements. You should be quantifying this, using measurement tools to collect data.

### 4. Conclusion

Reflect on your project. Some questions to consider: What did you learn from it? What went well? What accomplishments are you proud of? What did you learn/gain from this experience? Did you have to change your approach? What could have been done differently? Did you encounter obstacles that you didn’t anticipate? What could be a next step for this project?

## MVP Demo

1.github URL 

upenn-embedded/final-project-v50: ese5190f24-final-project-final-project-skeleton created by GitHub Classroom 

2. 

 

3. 

We use two separate ADCs to simultaneously acquire data, and then utilize DMA to store the data in arrays for further processing. 

void ADC1_DMA_Init(void) { 

    // Enable clocks for ADC1 and DMA2 

    RCC->APB2ENR |= RCC_APB2ENR_ADC1EN; // Enable ADC1 clock 

    RCC->AHB1ENR |= RCC_AHB1ENR_DMA2EN; // Enable DMA2 clock 

 

    // Configure GPIOA pin PA0 (ADC1_IN0) as analog mode 

    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN; // Enable GPIOA clock 

    GPIOA->MODER |= (3 << (0 * 2));      // Set PA0 to analog mode 

 

    // Configure ADC1 

    ADC1->CR1 = 0;                       // 12-bit resolution, single channel 

    ADC1->CR2 = ADC_CR2_CONT |           // Continuous conversion mode 

                ADC_CR2_DMA |            // Enable DMA 

                ADC_CR2_DDS;             // DMA in circular mode 

    ADC1->SQR3 = 0;                      // Channel 0 (ADC1_IN0) 

    ADC1->SMPR2 = (7 << 0);              // Set sample time for channel 0 (480 cycles) 

 

    // Configure DMA2 Stream 0 (connected to ADC1) 

    DMA2_Stream0->CR = 0;                // Reset control register 

    while (DMA2_Stream0->CR & DMA_SxCR_EN); // Ensure DMA stream is disabled 

 

    DMA2_Stream0->PAR = (uint32_t)&ADC1->DR;     // Peripheral address (ADC1 data register) 

    DMA2_Stream0->M0AR = (uint32_t)adc1_val; // Memory address (buffer) 

    DMA2_Stream0->NDTR = ADC_BUFFER_SIZE;       // Number of data to transfer 

    DMA2_Stream0->CR = DMA_SxCR_PL_1 |          // High priority 

                       DMA_SxCR_MSIZE_0 |       // Memory size: 16-bit 

                       DMA_SxCR_PSIZE_0 |       // Peripheral size: 16-bit 

                       DMA_SxCR_MINC |          // Memory increment mode 

                       DMA_SxCR_CIRC |          // Circular mode 

                       DMA_SxCR_DIR_0;          // Peripheral-to-memory direction 

 

    // Enable DMA2 Stream 0 

    DMA2_Stream0->CR |= DMA_SxCR_EN; 

 

    // Enable ADC1 

    ADC1->CR2 |= ADC_CR2_ADON; 

 

    // Start ADC1 conversion 

    ADC1->CR2 |= ADC_CR2_SWSTART; 

} 

 

We configure one of the ADC channels as an analog input and use DMA to transfer the data to the adc1_val buffer. The same procedure is applied to the second ADC. 

Once we collect 256 data points in both the adc1_val and adc2_val buffers, we compare the values. Using the position relationship, we employ a counter to determine which buffer has the higher value. If the adc1_val contains the larger value, we set PA5 high for 80ms, then set it low. If the adc2_val contains the larger value, we set PA6 high for 80ms, then set it low. These signals are connected directly to the ATMEGA328PB, which uses them to determine the motor's rotation direction. 

int adc1_greater_count = 0; 

 

        for (int i = 0; i < ADC_BUFFER_SIZE; i++) { 

            if (ADC1_Buffer[i] > ADC2_Buffer[i]) { 

                adc1_greater_count++; 

            } 

        } 

 

        if (adc1_greater_count > (ADC_BUFFER_SIZE / 2)) { 

            GPIOA->ODR |= GPIO_ODR_OD5;  

            HAL_Delay(80); 

            GPIOA->ODR &= ~GPIO_ODR_OD5; 

        } else { 

            GPIOA->ODR |= GPIO_ODR_OD6;  

            HAL_Delay(80); 

            GPIOA->ODR &= ~GPIO_ODR_OD5; 

        } 

 

In addition, we use ARM math libraries to perform a Fast Fourier Transform (FFT) on the adc1_val buffer to analyze the frequency content. The resulting data is sent via UART to the ATMEGA328PB, which interfaces with an LCD module to display the frequency information. 

        init_data(); 

      arm_cfft_f32(&arm_cfft_sR_f32_len256 , fft_data, 0 ,1); 

      arm_cmplx_mag_f32(fft_data, data_out, DATA_LENGTH / 2); 

      avg[0] = 0; 

      for(int i = 0; i < 25; i++) { 

          avg[0] += data_out[i]; 

      } 

      avg[0] /= 128; //100Hz 

 

      avg[1] = 0; 

      for(int i = 25; i < 50; i++) { 

          avg[1] += data_out[i]; 

      } 

      avg[1] /= 128; //200Hz 

 

      avg[2] = 0; 

      for(int i = 50; i < 75; i++) { 

          avg[2] += data_out[i]; 

      } 

      avg[2] /= 128; //300Hz 

 

      avg[3] = 0; 

      for(int i = 75; i < 100; i++) { 

          avg[3] += data_out[i]; 

      } 

 

 

      avg[4] = 0; 

      for(int i = 100; i < 125; i++) { 

          avg[4] += data_out[i]; 

      } 

      avg[4] /= 128; //500Hz 

 

      for(int i = 0; i < 5; i++){ 

          char* buffer; 

          float_to_string(avg[i], buffer, 1); 

          UART_SendString(buffer); 

      } 

 

We use two seperate ports as input signal to receive signal to determine whether motor rotate left or right. 

void gpio_init() { 

    DDRD &= ~(1 << LEFT_PIN); 

    DDRD &= ~(1 << RIGHT_PIN); 

    DDRB |= (1 << LED); 

} 

 

 

For motors, we use 280Hz frequency to control our motor, and vary its duty cycle by changing OCR0B. 

void timer1_init() {  

    // Set PB2 as output 

    DDRB |= (1 << MOTOR_PIN); 

    OCR1A = 900; // FREQUENCY 

    OCR1B = 450; // DUTY CYCLE = (OCR0A - OCR0B) / OCR0A 

     

    // Set Timer to FAST PWM mode with TOP = OCR0A (WGM02:0 = 001) 

    TCCR1A |= (1 << WGM10);  

    TCCR1A |= (1 << WGM11);   

    TCCR1B |= (1 << WGM12);  

    TCCR1B |= (1 << WGM13); 

     

    TCCR1A |= (1 << COM1A0); 

    TCCR1A |= (1 << COM1A1); 

     

    TCCR1A |= (1 << COM1B0); 

    TCCR1A |= (1 << COM1B1); 

 

    // Set prescaler to 256 

    TCCR1B &= ~(1 << CS12); 

    TCCR1B |= (1 << CS11); 

    TCCR1B |= (1 << CS10); 

 

} 

 

void rotate(uint8_t flag){ 

    if(flag == 1){ 

        if(OCR1B > 100){ 

            OCR1B--; 

        } 

    } 

    else{ 

        if(OCR1B <= 800){ 

            OCR1B++; 

        } 

    } 

} 

 

If signal PD4 received we let it rotate left, otherwise, rotate left. 

        if(PIND & (1 << PIND4)) { 

            rotate(0); 

            //PORTB |= (1 << LED); 

        } 

        else if(PIND & (1 <<PIND3)) { 

            //PORTB ^= (1 << LED);  

            rotate(1); 

            //PORTB &= ~(1 << LED); 

        } 

 

4. 

Currently we have make the microphone rotate accurate enough to follow the sound in 30cm 

 

5. We let the motor rotate as we expected to follow the sound. 

6. We still need to finish UART communication through ATMEGA328PB and STM32 and LCD module, also the 3D printing 

7. 

https://drive.google.com/file/d/1KB4CcpbpYN1mqRFmcyycBe9k0Av-pOrb/view?usp=sharing 

8. The riskiest part of the project is the microphone is not accurate enough, so need to manually calibrate it. 

   

## Sprint review #2

### Current state of project
We have finished the microphone and some fft algorithm, our next step is assemble our parts, since the microphone is not accurate enough, we are thinking using software to calibrate it.
### Last week's progress
Finished the sensor parts, and fft part, also we use 3D printing
### Next week's plan
assemble our part, finished UART to allow STM32 and ATMEGA communication, and LCD part(hopefully we can use LCD to demo every 100Hz of our voice signal.
## Sprint review #1

### Current state of project
2.
Since our components are not arrived, our main address for the week is set up the environment to do the work and to finish ADC module and open TIMER in stm32f4 to control the motor.
![image](https://github.com/user-attachments/assets/b2dfb0ca-b817-4143-8f69-2c955ac703e7)
4.
For the software part, due to the accuracy of the microphone, we can not too far away from the microphone.
5.
For the hardware part, UART and LCD still not finished.
6.
For the remaining of the process, we will finish LCD and UART part, also finish 3d printing.

8. the riskiest part of the project is component selection, we find the microphone data is not accurate that we expected, which means we can not too far away from the microphone 

### Last week's progress 
Finished components ordering, and have the ideal of the project.
### Next week's plan
Get the motor rotate with the sound.

## Sprint review trial

### Current state of project
### Last week's progress 
### Next week's plan

## Final Project Proposal

### 1. Abstract

This project aims to create a system that captures sound, then use motor platform to follow the source of the sound. The audio input will be processed by DSP to analyze its frequency components, which enable LEDs to respond to different frequencies. Additionally, the processed sound data will be sent to a server via Wi-Fi for further analysis and storage. 

### 2. Motivation

This project tackles two main challenges: digital signal processing (DSP) for noise filtering and motor control using PID (Proportional-Integral-Derivative) and PWM (Pulse Width Modulation) techniques. The objective is to develop a system that captures sound, particularly from objects emitting specific frequencies, and accurately tracks their source. By analyzing the frequency components of the sound, the system can isolate and follow a target, such as a drone. This capability has practical applications in real-life scenarios, such as helping individuals evade drones by detecting their direction. The integration of frequency analysis, noise filtering, and precise motor control makes this project both innovative and relevant.

### 3. Goals

This project aims to create a system that effectively tracks sound sources using multiple technologies. Our goals include:

1. Integrating three microphones to capture analog sound, converting it to discrete digital information using an ADC.
2. Comparing the amplitude of the captured sound to control a motor that tracks the sound source.
3. Implementing PID (Proportional-Integral-Derivative) control for precise motor movement.
4. Applying digital signal processing (DSP) algorithms to filter noise and separate different frequency components.
5. Utilizing DSP data to drive LED PWM lighting that visually responds to sound.
6. Sending processed data to a Wi-Fi module for further analysis and storage.
7. Ensuring a reliable power supply for the entire system.

### 4. System Block Diagram

![alt text](img1.png)

### 5. Design Sketches

![alt text](img2.png)

### 6. Software Requirements Specification (SRS)
### 6.1 Overview
This project creates a system that converts analog sound into digital data using the STM32F4 microcontroller. The data is transmitted to a server via Wi-Fi every few seconds. It uses digital signal processing (DSP) to filter noise and controls a motor with feedback from an MPU for PID control, allowing precise tracking of sound sources in real time.
### 6.2 Definitions, Abbreviations
Keil for develop the system.
MATLAB for frequnecy analysis and filter design. 
sound_data_before_filter 16 bits data after ADC
dsp_data data(bits to be determined) processed after DSP
pos_x MPU data(16 bits) send to MCU
pos_y MPU data(16 bits) send to MCU
### 6.3 Functionality
  1.SRS 01  The IMU 3-axis acceleration will be measured with 16-bit depth every 100 milliseconds +/-10 milliseconds
  2.SRS 02  The ADC value will be measured with 16-bit depth every 0.1 milliseconds
  3.SRS 01  The DSP data will be measured with n-bit depth(to be confirmed) every 100 milliseconds

### 7. Hardware Requirements Specification (HRS)
### 7.1 Overview
There will be 7 parts, microphone, motor, MCU, MPU, power supply, WIFI module, LED.
### 7.2 Functionality
The motors should rotate between 0 to 270 degrees and also when the sound is from center of the microphone, it should stay at the position, when left microphone detect the sound, it should rotate left, when right microphone detect the sound, it should rotate right. 
The LCD should show frequency components of 100 - 1200Hz.
Projects shall be based on STM32F401RE
microphone will need have good front, end pickup pattern, the initial thought would be KY-038
IMU shall be use MPU6050
WIFI module shall be use ESP32 
motor shall support IIC or CAN communication

### 8. Components

The MCU should have FPU and have a large relative RAM to complete DSP algorithms, and also have at least 2 ADC and one timer, our choice is STM32F446 and servo motor control, we use atmega328pb for motor control 
https://www.digikey.com/en/products/detail/stmicroelectronics/NUCLEO-F446RE/5347712

For microphone, we want to use module to reduce workload, our choice is KY-038, since it it modular, which reduce workload to redesign an amplifier
[https://www.amazon.com/DEVMO-Microphone-Sensitivity-Detection-Arduino/dp/B07R452F6J/ref=pd_lpo_sccl_2/147-7977676-9188223?pd_rd_w=YKrNA&content-id=amzn1.sym.4c8c52db-06f8-4e42-8e56-912796f2ea6c&pf_rd_p=4c8c52db-06f8-4e42-8e56-912796f2ea6c&pf_rd_r=00BK44ZNARHGD4G26S2C&pd_rd_wg=Z2npc&pd_rd_r=65de2fe2-2fd5-4124-b148-56c567dcddf5&pd_rd_i=B07R452F6J&th=1](https://www.amazon.com/KY-038-Sensitivity-Detection-Microphone-Module/dp/B09W157963)

For IMU, we will use MPU6050
https://www.digikey.com/en/products/detail/dfrobot/SEN0142/6588492

For Motors, we decide to use this servo motor
https://www.amazon.com/Miuzei-Waterproof-Compatible-Steering-Horn（270°）/dp/B0C5LWHTQ1/ref=sr_1_4_sspa?dib=eyJ2IjoiMSJ9.uAVrkFDoXjC5ivnbOqxanhmnWQfgWObJWhZ6x2ZaCcRzWP6Rmi_fvGBDKueZcs4yqv4MUuX8KGyA6zj8MhNSiweX9l7yT7uUVYyVMapjdSsnpJ7Pt9cJVlHWPMrgHJITVGxgH92vFTdJ9D56dI6DvlHON9QKuX-TJZ_OWkk12PXDx1NtCuKaBpUr95ZzUpHPz4bwhErdtMGJjL02d2gmFgdmv6ohiWK6MpyNG8glx5TcZh4s0jomp3VRq_GoKSZc-B_UvI8G0BskevWhi-aRKEQPIm4XGnT9E_YQEfitzX4.mSR_-otcXv9I-rw31MgP71H1RdjaBrz9xr1Vjfv_v0I&dib_tag=se&keywords=SERVO+Motor&qid=1730410329&sr=8-4-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1

### 10. Final Demo

For basic level of our system, it can detect sound and let the motor rotate to the source of the sound, if we can achieve advanced feature, we will let ESP32 to allow phone choose certain frequency and we only track that frequency of the sound.

### 10. Methodology

We will divide our task into 4 parts, first part is quite simple, that is get value from our sensors, second is let motor rotate with degree that we want, the third part also the most chanllenging part, we need to complete a DSP algorithm and let LED shining, fourth part is our motor with PID control and keep track of people. 
we will set first task and second task a short deadline, then for the third and fourth we will assign a longer deadline.

### 11. Evaluation
The outcome of this project will be a functional prototype that serves as an initial proof of concept. We will conduct thorough testing to evaluate the performance of both the DSP filtering and the PID control mechanisms. Key evaluation criteria will include:

Performance Metrics: Assessing the accuracy and stability of the motor control, along with the effectiveness of the filter in reducing noise.

### 12. Proposal Presentation

Add your slides to the Final Project Proposal slide deck in the Google Drive.

### 13. Sprint 1(first week 11.8)

   * Progress over the past week:  
     Buy all of the needed component(upload the image in first week file)     
     Write the ADC libarary(upload in ADC.file)    
   *Overall status of the project:  
     Buy all of the needed component including backup(upload the image in first week file)  
     Finish all of the prepared step  
   *Plans for next week:  
     Before 11.15 we will test the ADC function of the STM32 board, and the communication between ATmegas  
     Get the motor rate of the sound.  

### 14 Sprint 2(second week 11.15)  
 * Progress over the past week:  
      1.We tested the function of the microphone and found that the KY038 is not sensitive to the external ring sound,
      and it is not possible to adjust the amplification, and it can only present a positive value in the output,
      which does not satisfy the function of our device.
   
      2. On the code side, the ADC code for the microphone was completed,
      the control algorithm for the motor, and the microphone algorithm used bandpass filtering.

   *Overall status of the project:
      Approximately 50 percent of the project is completed
   
   *Plans for next week:
   Purchase a new microphone COMPONENT and test the new microphone to see if it can be realized to detect the volume of the environment.
   Finish porting the code on stm32.
     
## References

Fill in your references here as you work on your proposal and final submission. Describe any libraries used here.



