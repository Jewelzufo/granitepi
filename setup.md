# Running IBM Granite 3.2-2B on Raspberry Pi 5: Simple CPU Setup Guide

>**Author:** 
>*Julian A. Gonzalez*, **IBM Champion 2025**
>
>**Date:** 
>*6-29-2025*

---

<div style="display: flex; justify-content: left;">

  <img src="https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Follama.com%2Fpublic%2Fblog%2Follama_ibm.png&f=1&nofb=1&ipt=cc667c105ab0fcbf2af612f457a6c6d2c3d43e1d74ea2c6670ffacdef379f521" alt="Image 1" height="150" width="250"/>
  
  <img src="https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fwewalab.com%2Fwp-content%2Fuploads%2F2017%2F08%2FRaspberry-Pi-Logo-01-768x768.png&f=1&nofb=1&ipt=98da33f36784e17bedd25eed2821d635df23913b10f0a3fdacd183813e0ef280" alt="Image 2" height="200" width="200"/>
</div>


## Introduction
Welcome to this beginner-friendly guide for running IBM's Granite 3.2-2B language model on a Raspberry Pi 5! This guide shows you how to set up a local AI assistant using only your Raspberry Pi's built-in hardware - no external drives or complex configurations needed.

**What to Expect**:  
The Granite 3.2-2B model has 2 billion parameters and runs well on the Raspberry Pi 5's CPU. Expect responses in 5-15 seconds. Your Pi will get warm during use, so good ventilation is recommended.

## What You Need
### Required Hardware
- Raspberry Pi 5 (8GB RAM strongly recommended, 4GB minimum)
- MicroSD card (64GB or larger, Class 10 speed)
- Official power supply (27W USB-C recommended)
- Cooling (case with fan or heatsink recommended)
- Monitor, keyboard, and mouse for setup
- Internet connection for downloading the model

### Optional but Recommended
- Active cooling case (keeps your Pi cool during AI processing)
- Ethernet connection (faster than Wi-Fi for initial setup)

> **Important**: This guide uses only the Raspberry Pi 5's built-in storage and memory. No external drives needed!

## Table of Contents
1. [Setting Up Your Raspberry Pi 5](#setting-up-your-raspberry-pi-5)
2. [Installing Ollama](#installing-ollama)
3. [Basic Performance Optimization](#basic-performance-optimization)
4. [Downloading Granite 3.2-2B](#downloading-granite-32-2b)
5. [Testing Your Setup](#testing-your-setup)
6. [Simple Usage Examples](#simple-usage-examples)
7. [Troubleshooting](#troubleshooting)
8. [What's Next](#whats-next)

## Setting Up Your Raspberry Pi 5
### Step 1: Install the Operating System
1. Download Raspberry Pi Imager from [rpi.org](https://rpi.org)
2. Flash your SD card:
   - Insert your microSD card into your computer
   - Open Raspberry Pi Imager
   - Choose "Raspberry Pi OS (64-bit)" - **Must be 64-bit!**
   - Select your SD card
   - Click the gear icon for advanced options:
     - Set a username and password
     - Enable SSH if you want remote access
     - Configure Wi-Fi if needed
   - Click "Write" and wait for it to finish
3. First boot:
   - Insert the SD card into your Raspberry Pi 5
   - Connect monitor, keyboard, mouse, and power
   - Follow the setup wizard

### Step 2: Update Your System
Open a terminal and run these commands:
```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

After reboot, verify you have the 64-bit system:
```bash
uname -m
```
You should see `aarch64` (this means 64-bit ARM).

## Installing Ollama
Ollama makes it easy to run AI models on your Raspberry Pi. Here's how to install it:

### Simple Installation
Run this single command to install Ollama:
```bash
curl -fsSL https://ollama.com/install.sh | sh
```
That's it! Ollama is now installed and running.

### Verify Installation
Check if Ollama is working:
```bash
ollama --version
```
You should see version information displayed.

## Basic Performance Optimization
Let's make a few simple changes to get the best performance from your Pi's CPU:

### Step 1: Set CPU to Performance Mode
This makes your Pi's CPU run at full speed:
```bash
echo "performance" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

### Step 2: Make Performance Mode Permanent
Create a simple service to set performance mode on every boot:
```bash
sudo nano /etc/systemd/system/cpu-performance.service
```
Copy and paste this content:
```plaintext
[Unit]
Description=Set CPU to Performance Mode
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
Save the file (`Ctrl+X`, then `Y`, then `Enter`), then enable it:
```bash
sudo systemctl enable cpu-performance.service
sudo systemctl start cpu-performance.service
```

### Step 3: Optimize Ollama for CPU
Tell Ollama to use all 4 CPU cores:
```bash
sudo systemctl edit ollama.service
```
Add these lines in the editor that opens:
```plaintext
[Service]
Environment=OLLAMA_NUM_THREADS=4
```
Save and exit, then restart Ollama:
```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

## Downloading Granite 3.2-2B
Now for the exciting part - getting the AI model!

### Method 1: Direct Download (Easiest)
Try this first - it might work directly:
```bash
ollama pull granite3.2:2b
```
If this works, skip to the Testing section.

### Method 2: Custom Setup (Recommended)
If the direct method doesn't work, we'll create a custom setup:
1. Create a model configuration file:
   ```bash
   nano granite-pi.Modelfile
   ```
2. Add this content (copy and paste):
   ```plaintext
   FROM ibm/granite-3.2-2b
   PARAMETER temperature 0.7
   PARAMETER num_ctx 2048
   SYSTEM """
   You are a helpful AI assistant running on Raspberry Pi 5 hardware.
   Keep responses concise and efficient due to hardware limitations.
   """
   ```
3. Create the model:
   ```bash
   ollama create granite-pi -f granite-pi.Modelfile
   ```
4. Verify the model is ready:
   ```bash
   ollama list
   ```

## Testing Your Setup
Let's make sure everything works!

### Basic Test
Start a conversation with your AI:
```bash
ollama run granite-pi
```
Try asking it something simple:
- `Hello! What can you help me with?`
- `What is a Raspberry Pi?`
- `Tell me a short joke`

To exit the conversation, type `/bye` or press `Ctrl+C`.

### Monitor Temperature (Optional)
Create a simple temperature monitor to keep an eye on your Pi:
```bash
cat > ~/check_temp.sh << 'EOF'
#!/bin/bash
temp=$(vcgencmd measure_temp | cut -d= -f2 | cut -d\' -f1)
echo "CPU Temperature: $temp"
if (( $(echo "$temp > 70" | bc -l) )); then
    echo "⚠️  Getting warm! Consider better cooling."
fi
EOF

chmod +x ~/check_temp.sh
```
Run it anytime with:
```bash
~/check_temp.sh
```

## Simple Usage Examples
Here are some easy ways to use your new AI assistant:

### Quick Questions
```bash
# Ask a quick question
echo "Explain what machine learning is in simple terms" | ollama run granite-pi --format json | jq -r '.response'
```

### Create a Simple Chat Script
```bash
cat > ~/chat.sh << 'EOF'
#!/bin/bash
echo "🤖 Granite AI Assistant (type 'quit' to exit)"
echo "=================================="

while true; do
    echo -n "You: "
    read input
    
    if [ "$input" = "quit" ]; then
        echo "Goodbye!"
        break
    fi
    
    echo -n "AI: "
    echo "$input" | ollama run granite-pi --format json | jq -r '.response'
    echo ""
done
EOF

chmod +x ~/chat.sh
```
Use it with:
```bash
~/chat.sh
```

### Check System Status
```bash
cat > ~/ai_status.sh << 'EOF'
#!/bin/bash
echo "🔍 Raspberry Pi AI Status"
echo "========================"
echo "Temperature: $(vcgencmd measure_temp)"
echo "Memory usage: $(free -h | grep Mem | awk '{print $3 "/" $2}')"
echo "CPU governor: $(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor)"
echo "Ollama status: $(systemctl is-active ollama)"
echo "Available models:"
ollama list
EOF

chmod +x ~/ai_status.sh
```

## Troubleshooting
### Problem: Model Won't Load
**Symptoms**: Error messages when trying to run the model  
**Solutions**:
1. Check available memory:
   ```bash
   free -h
   ```
2. Restart Ollama:
   ```bash
   sudo systemctl restart ollama
   ```
3. Try a smaller context size:
   ```bash
   ollama run granite-pi --num_ctx 1024
   ```

### Problem: Very Slow Responses
**Symptoms**: Responses take more than 30 seconds  
**Solutions**:
1. Check CPU temperature:
   ```bash
   vcgencmd measure_temp
   ```
2. Verify performance mode:
   ```bash
   cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
   ```
3. Close other applications to free up resources.

### Problem: Pi Becomes Unresponsive
**Symptoms**: System freezes or becomes very slow  
**Solutions**:
1. This usually means overheating. Improve cooling.
2. Consider using 8GB RAM model if using 4GB.
3. Reduce the model context size:
   ```bash
   ollama run granite-pi --num_ctx 1024
   ```

### Problem: "Out of Memory" Errors
**Symptoms**: Ollama crashes with memory errors  
**Solutions**:
1. Close all other applications.
2. Reboot your Pi to clear memory:
   ```bash
   sudo reboot
   ```
3. Consider upgrading to 8GB RAM model.

## What's Next?
Congratulations! You now have a working AI assistant on your Raspberry Pi. Here are some ideas for what to do next:

### Experiment with Different Prompts
Try asking your AI to:
- Write code snippets
- Explain complex topics simply
- Help with homework (but don't cheat!)
- Create stories or poems
- Answer questions about technology

### Create Useful Scripts
```bash
# Daily briefing script
cat > ~/daily_brief.sh << 'EOF'
#!/bin/bash
echo "Good morning! Here's your daily tech tip:" | ollama run granite-pi --format json | jq -r '.response'
EOF
```

### Learn More About AI
Your Raspberry Pi is a great platform for learning about:
- How AI models work
- Different types of AI tasks
- Programming with AI assistance
- Building AI-powered projects

### Join the Community
- [Raspberry Pi Forums](https://forums.raspberrypi.com)
- [Ollama GitHub](https://github.com/ollama/ollama)
- [IBM Granite Documentation](https://ibm.com/granite)

## Performance Expectations
| Raspberry Pi Model | Response Time     | Best Use Cases                     |
|--------------------|-------------------|------------------------------------|
| Pi 5 (4GB RAM)     | 10-20 seconds     | Simple questions, learning         |
| Pi 5 (8GB RAM)     | 5-15 seconds      | General use, longer conversations  |

## Tips for Best Performance
1. **Keep it cool**: Good cooling = better performance
2. **Close other apps**: Give your AI all available resources
3. **Use shorter prompts**: Longer questions take more time
4. **Be patient**: AI on a $100 computer is still amazing!

## Safety and Care
### Protecting Your Pi
- Always use proper cooling
- Don't run intensive AI tasks for hours without breaks
- Monitor temperature regularly
- Use a quality power supply

### Responsible AI Use
- Remember this AI can make mistakes
- Don't rely on it for critical decisions
- Verify important information from other sources
- Have fun but use common sense!

## Conclusion
You've successfully set up a powerful AI assistant using just your Raspberry Pi 5's built-in hardware! This simple setup proves that you don't need expensive equipment to experiment with AI technology.

The Granite 3.2-2B model running on your Pi's CPU provides a great introduction to local AI without any cloud dependencies. You have complete control over your data and can experiment freely.

Remember:
- Start with simple questions and gradually try more complex tasks
- Monitor your Pi's temperature to ensure good performance
- Have fun exploring what your AI assistant can do!
- Share your experiences with the Raspberry Pi community

This is just the beginning of your AI journey. Enjoy exploring the possibilities!

*Julian A. Gonzalez*, **IBM Champion 2025**
