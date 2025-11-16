# Network Optimization Recommendations

## Problem Statement
Wife experiencing bad network connectivity during concurrent Zoom meetings (10am-5pm). Both using MacBook Airs on WiFi for video calls.

## Current Status
- ✅ Fixed: Changed pasillo 2.4GHz from 40MHz to 20MHz channel width
- ✅ Confirmed: Both MacBooks using 5GHz network (office SSID)

## Priority Recommendations

### 1. Enable SQM/QoS (HIGH PRIORITY) ⭐

**Problem:** No traffic prioritization - gaming consoles, streaming, and other traffic can interfere with Zoom calls

**Solution:** Configure Smart Queue Management (SQM) on EdgeRouter

**Steps:**
1. Access OpenWRT web interface at `http://10.0.0.1`
2. Navigate to: System → Software
3. Install packages:
   - `sqm-scripts`
   - `luci-app-sqm`
4. Navigate to: Network → SQM QoS
5. Configure:
   - **Enable this SQM instance:** ✓
   - **Interface name:** [Need to identify WAN interface - check Network → Interfaces]
   - **Download speed:** 580000 kbit/s (580 Mbps - leave ~3% headroom)
   - **Upload speed:** 580000 kbit/s
   - **Queue discipline:** cake
   - **Queue setup script:** piece_of_cake.qos
   - **Link layer:** Ethernet with overhead
   - **Per-packet overhead:** 0 (or 44 if using PPPoE)
6. Save & Apply

**Expected Impact:** Prevents bufferbloat, ensures Zoom packets get priority over bulk downloads/uploads

### 2. Additional Zoom Traffic Prioritization (MEDIUM PRIORITY)

**Problem:** Even with SQM, explicit DSCP marking helps Zoom traffic

**Solution:** Add firewall rules to mark Zoom traffic as high priority

**Implementation:** Add to `/etc/firewall.user` via LuCI (Network → Firewall → Custom Rules):

```bash
# Mark Zoom UDP traffic (video/audio) as Expedited Forwarding
iptables -t mangle -A FORWARD -p udp --dport 8801:8810 -j DSCP --set-dscp-class EF
iptables -t mangle -A FORWARD -p udp --sport 8801:8810 -j DSCP --set-dscp-class EF

# Mark Zoom TCP traffic (control/screen sharing) as high priority
iptables -t mangle -A FORWARD -p tcp --dport 8801:8810 -j DSCP --set-dscp-class AF41
iptables -t mangle -A FORWARD -p tcp --sport 8801:8810 -j DSCP --set-dscp-class AF41

# Additional Zoom ports
iptables -t mangle -A FORWARD -p udp --dport 3478:3479 -j DSCP --set-dscp-class EF
iptables -t mangle -A FORWARD -p udp --sport 3478:3479 -j DSCP --set-dscp-class EF
```

**Expected Impact:** Ensures Zoom packets get explicit priority marking

### 3. Band Steering / 2.4GHz Management (MEDIUM PRIORITY)

**Option A: Disable 2.4GHz on "office" SSID**
- Pros: Forces 5GHz usage, eliminates 2.4GHz interference
- Cons: No fallback if 5GHz coverage is weak in some areas
- Recommendation: Only if 5GHz coverage is excellent throughout work areas

**Option B: Keep 2.4GHz with lower power**
- In Unifi Controller: Settings → Wireless Networks → office → Advanced
- Set 2.4GHz transmit power to "Low" or "Medium"
- Enable "Band Steering" if available
- Set minimum RSSI for 2.4GHz to -70 dBm (forces disconnect if signal too weak)

**Recommended:** Option B for flexibility

### 4. Optimize 5GHz Channels (MEDIUM PRIORITY)

**Current Configuration:**
- pasillo: 5GHz ch 48 (80 MHz)
- cocina: 5GHz ch 40 (80 MHz)

**Potential Issue:** Channels 40 and 48 with 80MHz width may overlap slightly

**Recommendation:** Check for interference using WiFi analyzer
- If both APs are in close proximity, consider:
  - pasillo: ch 36 (80 MHz)
  - cocina: ch 149 (80 MHz)
  - This creates maximum separation in the 5GHz band

**How to Change:**
- Unifi Controller (on Raspberry Pi)
- Devices → Select AP → Settings → Radio → 5GHz Channel

### 5. Bandwidth Limits on VLAN 20 (MEDIUM PRIORITY)

**Problem:** 4 gaming consoles + streaming devices on VLAN 20 could saturate connection during work hours

**Solution:** Set bandwidth limits on VLAN 20 during business hours (10am-5pm)

**Implementation in OpenWRT:**
1. Network → Firewall → Traffic Rules
2. Create bandwidth limit rules for VLAN 20 (10.0.20.0/24)
3. Or use time-based QoS policies

**Alternative:** Ask family to avoid heavy gaming/streaming during 10am-5pm

### 6. AP Roaming Optimization (LOW PRIORITY)

**Current Setup:** Two APs (pasillo and cocina) - MacBooks may roam between them

**Recommendation:** Optimize roaming settings in Unifi Controller
- Settings → Wireless Networks → Advanced
- **Minimum RSSI:** -70 dBm (disconnect clients with weak signal, forcing roam)
- **AP Selection:** Load balancing disabled during Zoom calls

**Note:** Only adjust if you notice connection drops during movement

### 7. Disable 2.4GHz on "office" SSID (OPTIONAL)

**Only if:** You're confident 5GHz coverage is excellent everywhere you work

**How to implement:**
- Unifi Controller → Settings → Wireless Networks → office
- Uncheck "Enable 2.4GHz" or set 2.4GHz to "Disabled"

## Monitoring System (Next Steps)

To identify the root cause of the connectivity issues, we need to build a monitoring system that tracks:

1. **Bandwidth utilization** per VLAN
2. **WiFi signal strength** (RSSI) on MacBooks during calls
3. **Channel utilization** on both APs
4. **Latency and packet loss** to Zoom servers
5. **Bufferbloat testing** (before and after SQM)

### Tools to deploy:
- **On Raspberry Pi:** iperf3 server, bandwidth monitoring scripts
- **On MacBooks:** WiFi diagnostics during calls
- **On EdgeRouter:** collectd or Prometheus for metrics
- **Grafana dashboard:** Visualize all metrics in real-time

## Implementation Priority

**Week 1 - Immediate Actions:**
1. ✅ Fix pasillo 2.4GHz channel width (DONE)
2. Install and configure SQM on EdgeRouter
3. Test bufferbloat before/after: https://www.waveform.com/tools/bufferbloat
4. Run concurrent Zoom test calls

**Week 2 - Monitoring:**
1. Deploy monitoring scripts on Raspberry Pi
2. Collect data during actual Zoom calls
3. Analyze patterns

**Week 3 - Fine-tuning:**
1. Adjust QoS settings based on monitoring data
2. Optimize WiFi channels if needed
3. Implement bandwidth limits on VLAN 20 if necessary

## Testing Checklist

After implementing SQM:
- [ ] Run bufferbloat test: https://www.waveform.com/tools/bufferbloat
- [ ] Run speedtest during idle: https://speedtest.net
- [ ] Run speedtest during simulated load (large download on another device)
- [ ] Test concurrent Zoom calls during light network usage
- [ ] Test concurrent Zoom calls during heavy usage (kids gaming/streaming)
- [ ] Monitor Zoom connection quality statistics during calls

## Additional Resources

**Zoom Ports to Monitor:**
- UDP: 3478-3479, 8801-8810
- TCP: 8801-8810
- See: https://support.zoom.us/hc/en-us/articles/201362683

**OpenWRT QoS Documentation:**
- https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm

**Unifi Network Best Practices:**
- https://help.ui.com/hc/en-us/articles/221029967-UniFi-Troubleshooting-Connectivity-Issues
