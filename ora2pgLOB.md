### **How to Improve AWS DMS Task Speed for LOB Data**  

AWS Database Migration Service (DMS) can slow down when migrating **Large Object (LOB) data** (e.g., BLOBs, CLOBs, XML). Below are key optimizations from the video, decoded and summarized:

---

## **1. LOB Handling Modes in AWS DMS**
AWS DMS provides **three LOB modes** to control migration behavior:

| **LOB Mode**      | **How It Works** | **Impact on Speed** | **Best For** |
|------------------|----------------|-------------------|------------|
| **Full LOB Mode** | Migrates entire LOBs in chunks (configured via **LOB Chunk Size**). DMS fetches LOBs one by one. | 🐢 **Slowest** (multiple round trips to source DB) | When **no data loss** is acceptable. |
| **Limited LOB Mode** | Truncates LOBs exceeding the **Max LOB Size** (default: 32 KB). DMS pre-allocates resources. | ⚡ **Faster** (bulk processing) | When small LOBs dominate and **truncation is acceptable**. |
| **Inline LOB Mode** *(CLI-only)* | Uses **Limited Mode** for small LOBs and **Full Mode** for large LOBs (based on `InlineLOBMaxSize`). | ⚡ **Balanced speed & completeness** | Best for mixed LOB sizes (requires CLI setup). |

---

## **2. How to Configure LOB Modes**
### **A. Full LOB Mode (Console)**
1. In the **DMS Console** → **Create Task** → **Task Settings**.  
2. Under **LOB Settings**, select **Full LOB Mode**.  
3. Set **LOB Chunk Size** (e.g., 64 KB).  
   - Smaller chunks = more round trips = slower.  
   - Larger chunks = more memory usage.  

### **B. Limited LOB Mode (Console)**
1. In **Task Settings**, select **Limited LOB Mode**.  
2. Set **Max LOB Size** (e.g., 64 KB).  
   - LOBs larger than this are **truncated** (check logs for warnings).  

### **C. Inline LOB Mode (AWS CLI Only)**
1. Create a **JSON file** (`task-settings.json`):  
   ```json
   {
     "TargetMetadata": {
       "LobMode": "Inline",
       "InlineLobMaxSize": 64  // Size in KB
     }
   }
   ```
2. Apply using AWS CLI:  
   ```bash
   aws dms modify-replication-task \
     --replication-task-arn "your-task-arn" \
     --replication-task-settings file://task-settings.json
   ```
   - **Note:** `InlineLOBMaxSize` only works if `FullLOBMode=true`.  

---

## **3. Best Practices to Speed Up DMS with LOBs**
✅ **Use Inline LOB Mode** (best balance of speed & completeness).  
✅ **Increase `BatchApplyEnabled`** (bulk writes reduce round trips).  
✅ **Tune `MaxFileSize` and `ParallelLoadThreads`** for S3 targets.  
✅ **Avoid Full LOB Mode** unless necessary (slow for large LOBs).  
✅ **Monitor CloudWatch Logs** for truncation warnings.  

---

## **4. Key Takeaways**
- **For small LOBs** → **Limited LOB Mode** (fastest).  
- **For large LOBs** → **Inline LOB Mode** (CLI required).  
- **For zero truncation** → **Full LOB Mode** (slowest).  



