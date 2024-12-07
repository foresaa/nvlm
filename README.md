<img src = "perpop.png" alt = "Performance Properties" style = "max-width: 50%; height: 100; display: block; margin: 20px auto;" />

# NVML GPU Dashboard and Processes Viewer

This repository provides two lightweight HTML pages, **dashboard.html** and **processes.html**, designed to give basic insights into NVIDIA GPU performance and processes. While simple in implementation, these tools lay the groundwork for potential expansion into real-time GPU monitoring services.

## Features

### **1. dashboard.html**

- Displays summary GPU statistics:
  - GPU Name
  - Total and Used Memory
  - GPU Utilization
  - Temperature
- Provides a simple interface to monitor key GPU metrics at a glance.
- Includes an image for better visual appeal.

### **2. processes.html**

- Lists all running processes with:
  - Process ID (PID)
  - Process Name
  - GPU Memory Usage (if applicable)
  - CPU Memory Usage
- Enables a detailed view of resource usage by processes that interact with the GPU.

---

## Limitations

### What These HTML Pages **Do Not** Do:

1. **Direct GPU Access:** These HTML files cannot directly access GPU hardware or query real-time metrics due to browser sandboxing and security restrictions.
2. **Real-Time Monitoring:** The data presented is static unless paired with a backend service to provide live updates.
3. **System-Specific Data:** The tool cannot gather detailed GPU data from users’ systems unless they run a compatible local Python backend.

---

## How to Use

### **1. Hosted on GitHub Pages**

You can access the hosted version of these pages at:

- **Dashboard**: [dashboard.html](https://foresaa.github.io/nvlm/dashboard.html)
- **Processes**: [processes.html](https://foresaa.github.io/nvlm/processes.html)

**Note**: The hosted version demonstrates the static structure and design of the tool. It does not provide real-time GPU statistics or process monitoring.

### **2. For Local Use with Real Data**

To use these HTML files with actual GPU data, you need to run a Python backend locally:

#### **Requirements**

- An NVIDIA GPU with the NVIDIA Management Library (NVML) installed.
- Python 3.x.
- The `pynvml` library for Python:
  ```bash
  pip install nvidia-ml-py3
  ```

#### **Setting Up the Python Backend**

1. Clone this repository:
   ```bash
   git clone https://github.com/foresaa/nvlm.git
   cd nvlm
   ```
2. Create a Python script called `app.py` with the following content:
   _(This provides an API for real-time GPU stats and processes.)_

   ```python
   from flask import Flask, jsonify
   import psutil
   import subprocess

   app = Flask(__name__)

   @app.route('/gpu-details', methods=['GET'])
   def gpu_details():
       gpu_info = subprocess.check_output(['nvidia-smi', '--query-gpu=name,memory.total,memory.used,utilization.gpu,temperature.gpu', '--format=csv,noheader'], encoding='utf-8')
       name, total_mem, used_mem, gpu_util, temp = map(str.strip, gpu_info.split(','))
       return jsonify({
           "GPU Name": name,
           "Total Memory": total_mem,
           "Used Memory": used_mem,
           "GPU Utilization": gpu_util,
           "Temperature": temp
       })

   @app.route('/processes', methods=['GET'])
   def gpu_processes():
       gpu_processes_raw = subprocess.check_output(['nvidia-smi', '--query-compute-apps=pid,name,used_memory', '--format=csv,noheader'], encoding='utf-8')

       gpu_usage = {}
       for line in gpu_processes_raw.strip().split('\n'):
           try:
               pid, name, gpu_memory = map(str.strip, line.split(','))
               gpu_usage[int(pid)] = {"name": name, "gpu_memory": gpu_memory}
           except ValueError:
               continue

       all_processes = []
       for proc in psutil.process_iter(['pid', 'name', 'memory_info']):
           try:
               pid = proc.info['pid']
               name = proc.info['name']
               cpu_memory = f"{proc.info['memory_info'].rss / 1024 ** 2:.2f} MB"
               gpu_memory = gpu_usage.get(pid, {}).get('gpu_memory', '0 MB')

               all_processes.append({"pid": pid, "name": name, "gpu_memory": gpu_memory, "cpu_memory": cpu_memory})
           except (psutil.NoSuchProcess, psutil.AccessDenied):
               continue

       return jsonify(all_processes)

   if __name__ == '__main__':
       app.run(debug=True)
   ```

3. Run the backend:
   ```bash
   python app.py
   ```
4. Open `dashboard.html` and `processes.html` in your browser. The HTML pages will fetch real-time GPU and process data from the API running on `http://localhost:5000`.

---

## Why Add a Python API?

The Python API enhances the HTML pages significantly:

### **1. Real-Time GPU Monitoring**

- The HTML pages are static by default. The Python API allows you to fetch real-time GPU stats like memory usage, temperature, and process details dynamically.

### **2. Access System-Level Data**

- Browsers cannot directly access GPU hardware due to sandboxing and security restrictions.
- The Python API interacts directly with the NVIDIA Management Library (NVML) or `nvidia-smi`, allowing access to detailed GPU metrics.

### **3. Expandability**

Adding a Python API creates room for future enhancements:

- **Alerts**: Notify users when GPU metrics exceed certain thresholds.
- **Historical Data**: Log GPU metrics over time for performance analysis.
- **Multi-GPU Support**: Monitor multiple GPUs simultaneously.

---

## Future Plans

I am considering extending these tools into a full-fledged real-time GPU monitoring service. Potential features include:

- Real-time data streaming without requiring a local backend.
- Integration with cloud services for remote GPU monitoring.
- Advanced filtering and alerts for GPU resource usage.

---

## Contributing

Contributions are welcome! Feel free to fork this repository and submit pull requests with enhancements or new features.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Contact

For questions or suggestions, feel free to reach out via [GitHub Issues](https://github.com/foresaa/nvlm/issues) or directly on the repository.

---

Let me know if you'd like further refinements!
