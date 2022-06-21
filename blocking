/* mbed Microcontroller Library
 * Copyright (c) 2019 ARM Limited
 * SPDX-License-Identifier: Apache-2.0
 */

#include "mbed.h"
#include "platform/mbed_thread.h"

#include <cstdio>
#include <ros.h>
#include <std_msgs/Int32MultiArray.h>
// Blinking rate in milliseconds
#define BLINKING_RATE_MS                                                    500

ros::NodeHandle  nh;

ros::Publisher *srf_pub;


DigitalOut trigger1(PB_14);
DigitalIn  echo1(PB_13);
DigitalOut trigger2(PA_9);
DigitalIn  echo2(PA_8);
DigitalOut trigger3(PB_4);
DigitalIn  echo3(PB_5);

DigitalOut myled(LED1); //monitor trigger
DigitalOut myled2(LED2); //monitor echo

int correction;
Timer sonar;

int getDistanceUltrasonic(DigitalOut trig, DigitalIn echo){
    int distance = 0;
    int counter_wait_echo_high = 0;
    bool is_fail = false;
    // trigger sonar to send a ping
    trig = 1;
    myled = 1;
    myled2 = 0;
    sonar.reset();
    wait_us(10.0);
    trig = 0;
    myled = 0;
//wait for echo high
    while (echo==0) {
        counter_wait_echo_high++;
        if(counter_wait_echo_high > 65000){
            is_fail = true;
            break;
        }
        // printf("wait echoo high %d\n", counter_wait_echo_high);
    };

    if(is_fail)
        return 9999;

    counter_wait_echo_high = 0;
    myled2=echo;
//echo high, so start timer
    sonar.start();
//wait for echo low
    while (echo==1) {
        // counter_wait_echo_high++;
        // printf("wait echoo low %d\n", counter_wait_echo_high);
    };
//stop timer and read value
    sonar.stop();
//subtract software overhead timer delay and scale to cm
    distance = (sonar.read_us()-correction)/58.0;
    myled2 = 0;
    // printf(" %d cm \n\r",distance);

    return distance;
}

int main()
{
    nh.initNode();
    srf_pub = new ros::Publisher("/stm32/srf", new std_msgs::Int32MultiArray);


    nh.advertise(*srf_pub);
    // Initialise the digital pin LED1 as an output

    sonar.reset();
    // measure actual software polling timer delays
    // delay used later in time correction
    // start timer
    sonar.start();
    // min software polling delay to read echo pin
    while (echo1==2) {};
    myled2 = 0;
    // stop timer
    sonar.stop();
    // read timer
    correction = sonar.read_us();
    printf("Approximate software overhead timer delay is %d uS\n\r",correction);

    while (true) {
        printf("%d\t%d\t%d\n", getDistanceUltrasonic(trigger1, echo1), getDistanceUltrasonic(trigger2, echo2), getDistanceUltrasonic(trigger3, echo3));
     
        thread_sleep_for(20);
    }
}
