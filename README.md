# Granite-Pi
>**Author**: Julian A. Gonzalez, IBM Champion 2025  
>**Date**: 6-29-2025

![IBM Granite + Raspberry Pi](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Follama.com%2Fpublic%2Fblog%2Follama_ibm.png&f=1&nofb=1&ipt=cc667c105ab0fcbf2af612f457a6c6d2c3d43e1d74ea2c6670ffacdef379f521)

**Overview:** A beginner-friendly guide to running IBM's Granite 3.2-2B language model on Raspberry Pi 5 using only built-in hardware. Create a local AI assistant with CPU-only processing.

## 🚀 Key Features
- Runs entirely on Raspberry Pi 5 CPU (no GPU/external drives)
- 5-15 second response times
- 2 billion parameter AI model
- Simple setup with Ollama framework
- Complete local processing (no cloud dependency)

## ⚙️ Hardware Requirements
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Raspberry Pi | Pi 5 (4GB) | Pi 5 (8GB) |
| Storage | 64GB Class 10 microSD | Higher speed card |
| Power Supply | Standard USB-C | 27W Official PSU |
| Cooling | Heatsink | Active cooling case |
| Network | Wi-Fi | Ethernet connection |

## 📥 Installation Steps
1. **Set up Raspberry Pi OS**
   - Flash 64-bit Raspberry Pi OS using Imager
   - Enable SSH and configure Wi-Fi during setup
   - Update system: `sudo apt update && sudo apt upgrade -y`

2. **Install Ollama**
   ```bash
   curl -fsSL https://ollama.com/install.sh | sh
   ```

3. **Optimize Performance**
   - Set CPU governor to performance mode
   - Configure Ollama to use all 4 CPU cores

4. **Download Granite Model**
   ```bash
   ollama pull granite3.2:2b
   # Alternative custom setup if needed
   ollama create granite-pi -f granite-pi.Modelfile
   ```

## 🧪 Basic Testing
Start interactive session:
```bash
ollama run granite-pi
```

Sample prompts:
- "What is a Raspberry Pi?"
- "Tell me a short joke"
- "Explain machine learning simply"
Exit with `/bye` or `Ctrl+C`

## 📊 Performance Expectations
| Pi Model | Response Time | Best Use Cases |
|----------|---------------|---------------|
| Pi 5 (4GB) | 10-20 seconds | Simple questions |
| Pi 5 (8GB) | 5-15 seconds | Longer conversations |

## 🛠️ Troubleshooting
| Issue | Solution |
|-------|----------|
| Slow responses | Check cooling, verify performance mode |
| Out of memory | Close apps, reboot, reduce context size |
| System freezes | Improve cooling, use 8GB model |
| Model won't load | Restart Ollama, check memory |

## 💡 Usage Ideas
- Create chat scripts: `~/chat.sh`
- Generate daily briefings
- Explain complex topics
- Write code snippets
- Answer tech questions

## 🔒 Safety Tips
- Monitor temperature regularly
- Use quality power supply
- Don't run intensive tasks for hours
- Verify critical information from other sources

> **Note**: The Pi will get warm during operation - ensure proper cooling!

Explore the possibilities of local AI on affordable hardware!

[Full Guide Details](setup.md) | [Ollama GitHub](https://github.com/ollama/ollama) | [IBM Granite Docs](https://ibm.com/granite)
