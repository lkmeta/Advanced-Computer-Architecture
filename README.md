# **Advanced Computer Architecture**
## **Meta Louis-Kosmas**
## **AEM : 9390**

## Εργαστήριο νο1

### 1) Ανάλυση του αρχείου **starter_se.py** και άντληση πληροφοριών.

- **Υπάρχουν τρία είδη CPU και είναι:**
   + atomic
   + minor
   + hpi
   
   αυτά προκύπτουν από το ακόλουθο σημείο:
   
```c
cpu_types = {
    "atomic" : ( AtomicSimpleCPU, None, None, None, None),
    "minor" : (MinorCPU,
               devices.L1I, devices.L1D,
               devices.WalkCache,
               devices.L2),
    "hpi" : ( HPI.HPI,
              HPI.HPI_ICache, HPI.HPI_DCache,
              HPI.HPI_WalkCache,
              HPI.HPI_L2)
}
```

- **Η συχνότητα λειτουργίας είναι 1 GHz και η τάση είναι στα 3.3V τα οποία φαίνονται παρακάτω:**

```c
# Create a voltage and clock domain for system components
        self.voltage_domain = VoltageDomain(voltage="3.3V")
        self.clk_domain = SrcClockDomain(clock="1GHz",
                                         voltage_domain=self.voltage_domain)
```

- **Επιπλέον, η δημιουργία του memory bus και του memory mode που παίρνει την τιμή "timing" προκύπτουν από εδώ:**

```c
# Create the off-chip memory bus.
        self.membus = SystemXBar()
        
# Create a cache hierarchy (unless we are simulating a
        # functional CPU in atomic memory mode) for the CPU cluster
        # and connect it to the shared memory bus.
        if self.cpu_cluster.memoryMode() == "timing":
            self.cpu_cluster.addL1()
            self.cpu_cluster.addL2(self.cpu_cluster.clk_domain)
        self.cpu_cluster.connectMemSide(self.membus)
```

- **Όσον αφορά τον cluster συμπεραίνουμε ότι έχουμε τάση 1.2V.**

```c
# Add CPUs to the system. A cluster of CPUs typically have
# private L1 caches and a shared L2 cache.
self.cpu_cluster = devices.CpuCluster(self,
                                      args.num_cores,
                                      args.cpu_freq, "1.2V",
                                      *cpu_types[args.cpu])
```

- **Η μνήμη cache έχει τιμή 64 bytes.**

```c
# Use a fixed cache line size of 64 bytes
    cache_line_size = 64
```

- **Η default τιμή της mem_type είναι DDR3_1600_8x8.**

```c
    parser.add_argument("--mem-type", default="DDR3_1600_8x8",
                        choices=ObjectList.mem_list.get_names(),
                        help = "type of memory to use")
```

- **Η default τιμή της mem_size είναι 2GB.**

```c
    parser.add_argument("--mem-size", action="store", type=str,
                        default="2GB",
                        help="Specify the physical memory size")
```


- Στο αρχείο **hello** που βρίσκεται στον φάκελο **hello_res** που ουσιαστικά είναι και η πρώτη προσομοίωση που τρέχουμε, χρησιμοποιούμε όλες τις default τιμές εκτός από το cpu-type την οποία επιλέγουμε "minor".  
**Οι υπόλοιπες default τιμές φαίνονται παρακάτω.**  
  
```c
    parser = argparse.ArgumentParser(epilog=__doc__)

    parser.add_argument("commands_to_run", metavar="command(s)", nargs='*',
                        help="Command(s) to run")
    parser.add_argument("--cpu", type=str, choices=list(cpu_types.keys()),
                        default="atomic",
                        help="CPU model to use")
    parser.add_argument("--cpu-freq", type=str, default="4GHz")
    parser.add_argument("--num-cores", type=int, default=1,
                        help="Number of CPU cores")
    parser.add_argument("--mem-channels", type=int, default=2,
                        help = "number of memory channels")
    parser.add_argument("--mem-ranks", type=int, default=None,
                        help = "number of memory ranks per channel")
   
```

### 2) Ανάλυση των αρχείων **stats.txt**,**config.ini** και **config.json**.

#### **a) Αρχικά, από το config.ini αρχείο συμπεραίνουμε τα εξής:**
   + το είδος CPU που χρησιμοποιήθηκε στην προσομοίωση είναι τύπου **minor**
   + η τάση του επεξεργαστή είναι 3.3V και η συχνότητα είναι 1GHz (διότι έχουμε 1000 ticks)
   + η τάση του cluster είναι 1.2V και η συχνότητα είναι 4GHz (διότι έχουμε 250 ticks)
   + η μνήμη cache έχει μέγεθος 64 bytes 
   + το mem_mode παίρνει την τιμή "timing"  
   Παρατήρηση: αναμενόμενες τιμές όπως αναφέρθηκαν στην ανάλυση του **starter_se.py** αρχείου.  
   Αυτά προκύπτουν από τα ακόλουθα σημεία:
   
   ```c
   [system.cpu_cluster.cpus] 
   type=MinorCPU
   ...
   time_sync_period=100000000000
   ...
   cache_line_size=64
   ...
   mem_mode=timing
   ...
   [system.clk_domain]
   type=SrcClockDomain
   clock=1000
   ...
   [system.cpu_cluster.clk_domain]
   type=SrcClockDomain
   clock=250
   ...
   [system.voltage_domain]
   type=VoltageDomain
   voltage=3.3
   ...
   [system.cpu_cluster.voltage_domain]
   type=VoltageDomain
   voltage=1.2
   ```
   
   
   **Συνεχίζοντας, από το config.json αρχείο συμπεραίνουμε ότι επαληθεύονται όσα είπαμε για το config.ini από τα παρακάτω σημεία:**
      
   ```c
   "cpus": [
                {
                    "type": "MinorCPU",
                    "cxx_class": "MinorCPU",
                    "name": "cpus",
   ...
   "time_sync_period": 100000000000,
   ...
   "cache_line_size": 64
   ...
   "mem_mode": "timing"
   ...
   clk_domain": {
            "type": "SrcClockDomain",
            "cxx_class": "SrcClockDomain",
            "name": "clk_domain",
            "path": "system.clk_domain",
            "clock": [
                1000
            ]
   ...
   "clk_domain": {
                "type": "SrcClockDomain",
                "cxx_class": "SrcClockDomain",
                "name": "clk_domain",
                "path": "system.cpu_cluster.clk_domain",
                "clock": [
                    250
                ]
            }
   ...
   "voltage_domain": {
            "type": "VoltageDomain",
            "cxx_class": "VoltageDomain",
            "name": "voltage_domain",
            "path": "system.voltage_domain",
            "eventq_index": 0,
            "voltage": [
                3.3
            ]
        }
   ...
   "voltage_domain": {
                "type": "VoltageDomain",
                "cxx_class": "VoltageDomain",
                "name": "voltage_domain",
                "path": "system.cpu_cluster.voltage_domain",
                "eventq_index": 0,
                "voltage": [
                    1.2
                ]
            }
   ```
   
   **Aπό το stats.txt αρχείο έχουμε επιπλέον πληροφορίες που επαληθεύουν κάποια από τα πρηγούμενα:** 
   + η τάση του επεξεργαστή είναι 3.2V και η συχνότητα είναι 1GHz (διότι έχουμε 1000 ticks)
   + η τάση του cluster είναι 1.2V και η συχνότητα είναι 4GHz (διότι έχουμε 250 ticks)  
   **Παρατήρηση: έχουμε διαφορά εδώ τον επεξεργαστή στην τάση από 3.3V που περιμέναμε πλέον έχουμε 3.2V.**  
   Αυτά προκύπτουν από τα ακόλουθα σημεία:
   
| stats.txt |   |   |
| --- |:---:|--- 
| sim_freq                                  | 1000000000000 | # Frequency of simulated ticks    
| system.clk_domain.clock                   | 1000          | # Clock period in ticks  
| system.cpu_cluster.clk_domain.clock       | 250           | # Clock period in ticks
| system.cpu_cluster.voltage_domain.voltage | 1.200000      | # Voltage in Volts
| system.voltage_domain.voltage             | 3.300000      | # Voltage in Volts

   
   
#### **b) Aπό το stats.txt αρχείο βρήκαμε το σύνολο των "committed" εντολών όπως φαίνεται παρακάτω:**
   + έχουμε συνολικά 5028 εντολές σε 0.000024 δευτερόλεπτα που προσομοιώθηκαν  
   **Παρατήρηση: δεν υπάρχει διαφορά στα committed insts με τα simulated insts.**  
   Αυτά προκύπτουν από τα ακόλουθα σημεία:
   
| stats.txt |   |   |
| --- |:---:|---    
| sim_insts                             | 5028     | # Number of instructions simulated 
|system.cpu_cluster.cpus.committedInsts | 5028     | # Number of instructions committed
| sim_seconds                           | 0.000024 | # Number of seconds simulated  


#### **c) Aπό το stats.txt αρχείο βρήκαμε τον ζητούμενο αριθμό προσπελάσεων της l2 όπως φαίνεται παρακάτω:**
   + συγκεκριμένα, η l2 προσπελάστηκε συνολικά 479 φορές  

| stats.txt |   |   |
| --- |:---:|---    
| system.cpu_cluster.l2.overall_accesses::.cpu_cluster.cpus.inst | 332 | # number of overall (read+write) accesses
| system.cpu_cluster.l2.overall_accesses::.cpu_cluster.cpus.data | 147 | # number of overall (read+write) accesses
| system.cpu_cluster.l2.overall_accesses::total                  | 479 | # number of overall (read+write) accesses





