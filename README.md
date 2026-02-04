# **Multi‑Area OSPFv2 Practice Lab (CML 2.9.1)**  
A complete, scenario‑driven OSPFv2 lab built for Cisco Modeling Labs (CML 2.9.1), including:

- Multi‑area OSPF (Areas 0, 1, 2, 3)  
- Stub & Totally Stub areas  
- NSSA & Totally NSSA  
- ABR behavior  
- ASBR redistribution  
- DR/BDR election on broadcast segment  
- Virtual link operations  
- Full router configs embedded in the YAML (CML 2.9.1 “Extract Configs” feature)

This repository is designed for hands‑on practice, verification, and deep OSPF theory reinforcement.

---

# **📁 Repository Contents**

```
OSPF-Lab/
├── OSPF.yaml        # Full CML lab with embedded configs
└── README.md        # This documentation
```

---

# **🧩 Topology Overview**

The lab contains **10 routers** with the following area design:

| Router | Role | Areas | Notes |
|--------|------|--------|-------|
| R1 | Backbone | 0, 1 (VL endpoint) | DR on broadcast segment |
| R2 | Backbone | 0 | BDR on broadcast segment |
| R3 | ABR | 0, 1 | Connects Area 1 to backbone |
| R4 | Internal | 1 | Virtual‑link endpoint |
| R5 | ABR | 0, 2 | Stub area ABR |
| R6 | ABR | 0, 3 | NSSA ABR |
| R7 | Internal | 1 | Normal area |
| R8 | Internal | 2 | Stub area |
| R9 | ASBR | 3 | Redistributes static routes |
| R10 | Internal | 3 | NSSA internal |

### **Transit Networks**
- **10.0.100.0/24** — Broadcast segment for DR/BDR election (R1, R2, R3)
- **All other links** — Point‑to‑point /30

---

# **📘 Lab Scenarios**

This lab supports seven structured OSPF scenarios.  
Each scenario can be tested independently or sequentially.

---

## **1. Single‑Area OSPF (Area 0 Only)**

### **Objective**
- Bring up the backbone (R1–R2–R3–R5–R6)
- Validate neighbor formation and LSDB

### **Verification**
```
show ip ospf neighbor
show ip ospf database
show ip route ospf
```

### **Expected**
- All backbone routers FULL adjacency
- Type 1 & Type 2 LSAs only
- Full reachability to all loopbacks

---

## **2. Multi‑Area OSPF (Areas 0 + 1)**

### **Objective**
- Enable Area 1 on R3–R4–R7
- Observe ABR behavior

### **Verification**
```
show ip ospf
show ip ospf border-routers
show ip ospf database summary
```

### **Expected**
- R3 becomes ABR
- Type 3 LSAs injected into Area 1
- R4 & R7 see inter‑area routes (O IA)

---

## **3. Stub Area (Area 2)**

### **Objective**
- Configure Area 2 as stub (R5 ABR + R8 internal)
- Validate default route injection

### **Verification**
```
show ip ospf database external
show ip route
```

### **Expected**
- No Type 5 LSAs inside Area 2
- R8 receives default route `O*IA 0.0.0.0/0`

---

## **4. Totally Stub Area (Area 2)**

### **Objective**
- Convert Area 2 to totally stub
- Observe suppression of Type 3 LSAs

### **Config Notes**
- R5: `area 2 stub no-summary`
- R8: `area 2 stub`

### **Expected**
- Only intra‑area LSAs + default route

---

## **5. NSSA (Area 3)**

### **Objective**
- Configure Area 3 as NSSA
- Redistribute static routes on R9 (ASBR)

### **Verification**
```
show ip ospf database nssa-external
show ip ospf database external
```

### **Expected**
- Type 7 LSAs inside Area 3
- Type 5 LSAs outside Area 3 (ABR translation)

---

## **6. DR/BDR Election (Broadcast Segment)**

### **Objective**
- Validate DR/BDR election on 10.0.100.0/24

### **Expected Roles**
- **DR = R1** (priority 200)
- **BDR = R2** (priority 1)
- **DROTHER = R3** (priority 100)

### **Verification**
```
show ip ospf neighbor
```

### **Important Behavior**
OSPF **does not preempt**.  
If R3 joins after R2, R3 stays DROTHER even with higher priority.

---

## **7. Virtual Link (Area 1 Transit)**

### **Objective**
- Restore backbone continuity using a virtual link between R1 and R4

### **Requirements**
- R1 and R4 must be neighbors in Area 1  
- Area 1 must NOT be stub/NSSA  
- Router‑ID reachability must exist

### **Verification**
```
show ip ospf virtual-links
show ip ospf database router 1.1.1.1
show ip ospf database router 4.4.4.4
```

### **Expected**
```
Virtual Link OSPF_VL0 to router 4.4.4.4 is up
  State FULL
  Transit area 1
```

---

# **🧪 End‑to‑End Test Plan**

Use this checklist after loading the lab:

### **1. Adjacency Check**
```
show ip ospf neighbor
```
All routers should show FULL adjacency on their respective links.

### **2. LSDB Integrity**
```
show ip ospf database
```
Confirm:
- Type 1/2 in all areas  
- Type 3 from ABRs  
- Type 5 from ASBR (except stub areas)  
- Type 7 in NSSA  

### **3. Routing Table Validation**
```
show ip route ospf
```
Check:
- O routes (intra‑area)
- O IA routes (inter‑area)
- O N1/N2 (NSSA external)
- O E1/E2 (external)

### **4. Loopback Reachability**
From R1:
```
ping 2.2.2.2
ping 3.3.3.3
...
ping 10.10.10.10
```

### **5. Virtual Link**
```
show ip ospf virtual-links
```

---

# **📦 How to Use This Lab in CML 2.9.1**

### **1. Download the YAML**
```
OSPF.yaml
```

### **2. Import into CML**
- Home → Import Lab → Select YAML

### **3. Start All Nodes**

### **4. Verify Configs**
All configs are embedded using CML 2.9.1 “Extra Configs” feature.

---

# **📚 OSPF Theory Covered**

This lab reinforces:

- OSPF neighbor states  
- LSDB structure  
- SPF behavior  
- ABR/ASBR roles  
- LSA Types 1, 2, 3, 4, 5, 7  
- Stub, Totally Stub, NSSA, Totally NSSA  
- DR/BDR election  
- Virtual links  
- Summarization (optional extension)  
- Fast convergence tuning (optional extension)

---

# **📝 Notes**

- All interfaces use /30 point‑to‑point except the broadcast segment.  
- Loopbacks serve as router‑IDs and test prefixes.  
- The lab is intentionally modular so you can enable/disable scenarios without rewiring.

---

# **✔️ Final Thoughts**

This lab is built for **deep OSPF mastery**, not just configuration repetition.  
Every scenario forces you to validate LSDB behavior, routing decisions, and OSPF’s hierarchical design.

