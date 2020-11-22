# **Advanced Computer Architecture**
## **Meta Louis-Kosmas**
## **AEM : 9390**

## Εργαστήριο νο1

### 1) Ανάλυση του αρχείου **starter_se.py** και άντληση πληροφοριών.

- **Υπάρχουν τρία είδη CPUs και είναι:**
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
   
   
   **Συνεχίζοντας, από το config.json αρχείο επαληθεύονται όσα είπαμε για το config.ini από τα παρακάτω σημεία:**
      
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

### 3) Πληροφορίες για μοντέλα in-order CPUs και ανάπτυξη προγράμματος με gem5.

- **Υπάρχουν τρία μοντέλα in-orders SimpleCPUs και είναι τα εξής:**
   + BaseSimpleCPU
   + AtomicSimpleCPU
   + TimingSimpleCPU
   
   Αναλυτικότερα:
   ### + **BaseSimpleCPU**  
   The BaseSimpleCPU serves several purposes:  

    + Holds architected state, stats common across the SimpleCPU models.
    + Defines functions for checking for interrupts, setting up a fetch request, handling pre-execute setup, handling post-execute actions, and advancing the PC to the next instruction. These functions are also common across the SimpleCPU models.
    + Implements the ExecContext interface. The BaseSimpleCPU can not be run on its own. You must use one of the classes that inherits from BaseSimpleCPU, either AtomicSimpleCPU or TimingSimpleCPU.
   
   ### + **AtomicSimpleCPU**  
   The AtomicSimpleCPU is the version of SimpleCPU that uses atomic memory accesses. It uses the latency estimates from the atomic accesses to estimate overall cache access time. The AtomicSimpleCPU is derived from BaseSimpleCPU, and implements functions to read and write memory, and also to tick, which defines what happens every CPU cycle. It defines the port that is used to hook up to memory, and connects the CPU to the cache.
   
   ![AtomicSimpleCPU](https://www.gem5.org/assets/img/AtomicSimpleCPU.jpg)
   
   ### + **TimingSimpleCPU**  
   The TimingSimpleCPU is the version of SimpleCPU that uses timing memory accesses (see Memory systems for details). It stalls on cache accesses and waits for the memory system to respond prior to proceeding. Like the AtomicSimpleCPU, the TimingSimpleCPU is also derived from BaseSimpleCPU, and implements the same set of functions. It defines the port that is used to hook up to memory, and connects the CPU to the cache. It also defines the necessary functions for handling the response from memory to the accesses sent out.
   
   ![TimingSimpleCPU](https://www.gem5.org/assets/img/TimingSimpleCPU.jpg)
   
   Source: https://www.gem5.org/documentation/general_docs/cpu_models/SimpleCPU 
   

- **Εκτός από in-orders SimpleCPUs μοντέλα έχουμε και άλλα είδη όπως τα ακόλουθο:**

   ### - **Minor CPU Model**
   Minor is an in-order processor model with a fixed pipeline but configurable data structures and execute behaviour. It is intended to be used to model processors with strict in-order execution behaviour and allows visualisation of an instruction’s position in the pipeline through the MinorTrace/minorview.py format/tool. The intention is to provide a framework for micro-architecturally correlating the model with a particular, chosen processor with similar capabilities.
    
   Source: https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu
    

   
   
#### **a) Ο κώδικας που έγραψα για να δοκιμάσω την προσομοίωση κάνει το εξής:**
   + υπολογίζει τον fibonacci αριθμό του 46 με δυναμικό προγραμματισμό  
   
  Το πρόγραμμα γράφτηκε σε C και στην συνέχεια με χρήση της παρακάτω εντολής κάναμε compile για να μπορούμε να το τρέξουμε με τον gem5.
  ```c
  arm-linux-gnueabihf-gcc --static fib.c -o fib_arm
  ```  
  
  Εκτελώντας τις παρακάτω εντολές διαδοχικά, δημιουργήσαμε τις απαιτούμενες προσομοιώσεις για το πρόγραμμα μας.  
  Αναλυτικότερα, υλοποιούν το πρόγραμμα με χρήση δύο διαφορετικών CPUs με default επιλογή για όλα τα υπόλοιπα.  
  ```c
  ./build/ARM/gem5.opt -d fib_res_MinorCPU configs/example/se.py --cpu-type=MinorCPU --caches -c my_tests/fib_with_dynamic_programming/fib_arm
  ./build/ARM/gem5.opt -d fib_res_TimingSimpleCPU configs/example/se.py --cpu-type=TimingSimpleCPU --caches -c my_tests/fib_with_dynamic_programming/fib_arm
  ```  
   
   
   + **Χρόνοι εκτέλεσης προγράμματος**  
      + Βρίσκονται στο αρχείο **stats.txt** 
   
   | CPU model  |  sim_seconds |
   | --- |:---:|
   | Minor CPU | 0.000039 |
   | TimingSimpleCPU | 0.000047 |  
   
#### **b) Παρατηρώντας τα προηγούμενα, προκύπτει πως υπάρχει διαφορά χρόνων στα δυο διαφορετικά μοντέλα CPUs.**  
   
   Η διαφορά αυτή στους χρόνους που προέκυψαν οφείλεται στο γεγονός ότι ο TimingSimpleCPU περιμένει την προσπέλαση μνήμης να ολοκληρωθεί προτού συνεχίσει. Αντίθετα, στον MinorCPU δεν συμβαίνει αυτό με αποτέλεσμα να έχουμε ταχύτερη προσομοίωση.

#### **c) Αλλαγή μνήμης και συχνότητα λειτουργίας του προγράμματος.**  

Στο συγκεκριμένο ερώτημα έγιναν δύο αλλαγές που αφορούν την τεχνολογία της μνήμης και την συχνότητα ως εξής:  
+ **Αλλαγή μνήμης**  
   Τρέχοντας την παρακάτω εντολή,    
   ```c
   ./build/ARM/gem5.opt configs/example/se.py --list-mem-type
   ```
   
   προκύπτει η λίστα που ακολουθεί στην οποία φαίνονται όλες οι διαθέσιμες μνήμες που μπορούμε να χρησιμοποιήσουμε.  
      
   ![mem-list](https://github.com/lkmeta/Advanced-Computer-Architecture/blob/main/readme_imgs/mem_list.png "Mem List")
   
   Εμείς χρησιμοποιήσαμε την **DDR4_2400_16X4** για το παράδειγμα μας.
   
+ **Αλλαγή συχνότητας**  
   Η default συχνότητα που δοκιμάσαμε τo πρόγραμμα μας προηγουμένως ήταν 1GHz. Για το συγκεκριμένο ερώτημα ωστόσο, επιλέξαμε να τροποποιήσουμε την συχνότητα και να τρέξουμε δύο προσομοιώσεις με συχνότητες 0.8GHz και 1.4GHz. Αυτό υλοποιήθηκε με την προσθήκη των παρακάτω εντολών διαδοχικά.
   ```c
   --sys-clock="0.8GHz"
   --sys-clock="1.4GHz"
   ```
   
   
   Στις παρακάτω εικόνες φαίνονται οι εντολές που χρησιμοποιήθηκαν για το MinorCPU μοντέλο, αντίστοιχα έγιναν και για το TimingSimpleCPU.
   
   ![minorcpu_08](https://github.com/lkmeta/Advanced-Computer-Architecture/blob/main/readme_imgs/minorcpu_08.png "MinorCPU 0.8")
   
   ![minorcpu_14](https://github.com/lkmeta/Advanced-Computer-Architecture/blob/main/readme_imgs/minorcpu_14.png "MinorCPU 1.4")
   
   
   
 ### **Πίνακας με αποτελέσματα από τα αρχεία stats.txt**  
 
   | CPU model  |  sim_seconds | system.clk_domain.clock |
   | --- |:---:| ---: |
   | MinorCPU 0.8GHz | 0.000040 | 1250 |
   | MinorCPU 1.4GHz | 0.000036 |  714 |
   | TimingSimpleCPU 0.8GHz | 0.000048 | 1250 |
   | TimingSimpleCPU 1.4GHz | 0.000045 |  714 |
   
   Παρατηρούμε ότι οι χρόνοι στο TimingSimpleCPU μοντέλο παραμένουν μεγαλύτεροι από το MinorCPU μοντέλο, γεγονός που είναι αναμενόμενο. Επίσης λογικό και αναμενόμενο είναι η αύξηση της συχνότητας να μας οδηγεί σε μείωση του χρόνου σε κάθε μοντέλο ξεχωριστά.



## **Κριτική**

+ **Τι έμαθα από την εργασία;**  
Αρχικά, μέσα από τη συγκεκριμένη εργασία -η οποία ήταν και η πρώτη μου επαφή με την αρχιτεκτονική υπολογιστών- κατάφερα να κατανοήσω λίγο περισσότερο την δομή και την λειτουργία του επεξεργαστή. Σαφέστερα, θεωρώ πως πλέον έχω μια ελαφρώς διαφορετική αντίληψη για το πως ο επεξεργαστής εκτελεί εντολές ανάλογα με τι ορίσματα παίρνει σαν εντολές στην προσομοίωση. Η προσομοίωση με gem5 η οποία ήταν επίσης πρωτότυπη διαδικασία για εμένα, με βοήθησε να μάθω να χειρίζομαι κάποιες εντολές με σκοπό την διαχείριση των μοντέλων CPUs. Επιπλέον, μέσα από την συγκεκριμένη εργασία, έμαθα τα βασικά για Markdown και Github που δυστυχώς δεν γνώριζα νωρίτερα. Τέλος, εξοικειώθηκα περισσότερο με το VMware και τα Ubuntu.

+ **Θεωρώ την εργασία απλοϊκή ή δύσκολη και τι δυσκόλεψε;**  
Με βάση το επίπεδο γνώσεων μου στην αρχιτεκτονική υπολογιστών το οποίο είναι αρχάριο, θεωρώ πως η εργασία αυτή είναι σχετικά δύσκολη. Ακριβέστερα, βρέθηκα αντιμέτωπος με πολλά νέα σε εμένα προγράμματα και προβλήματα που απαίτηταν αρκετό χρόνο για την επίλυση τους. Επιπλέον, αρκετά σημεία με δυσκόλεψαν όπως το αρχικό build με την εντολή ``` scons build/ARM/gem5.opt -j Ν ``` στην οποία δεν ήξερα ακριβώς ποια τιμή θα έπρεπε να βάλω στο Ν. Ένα ακόμη σημείο που με δυσκόλεψε ήταν η εντολή ```  ./build/ARM/gem5.opt configs/example/se.py --cpu=MinorCPU –caches tests/test-progs/hello/bin/arm/linux/hello ``` η οποία για να τρέξει έπρεπε να τροποποιηθεί ως εξής: ``` ./build/ARM/gem5.opt configs/example/se.py --cpu=MinorCPU –-caches -c tests/test-progs/hello/bin/arm/linux/hello  ``` . Αυτά αποτέλεσαν δύο κύρια σημεία που καθυστέρησαν την ολοκλήρωση της εργασίας μου.

+ **Πως θα σχολίαζα τις οδηγίες που μας δόθηκαν;**  
Αρχικά, οι οδηγίες που δόθηκαν ήταν αναλυτικότατες και δεν υπήρχαν περιθώρια για να δημιουργηθούν απορίες. Αν εξαιρέσουμε τα δύο παραπάνω σημεία που με δυσκόλεψαν, όλα τα υπόλοιπα βγήκαν αμέσως ακολουθώντας τα βήματα από το pdf. Το μόνο που θα άλλαζα είναι οι διευκρινήσεις για το readme αρχείο, σε κάποια σημεία θεωρώ πως χρειαζόμουν περισσότερη καθοδήγηση για τις πληροφορίες που έπρεπε να πάρω από τα αρχεία των προσομοιώσεων. Επιπλέον, χρήσιμα και σημαντικά θεώρησα τις οδηγίες για την μετατροπή ενός αρχείου .c σε file που να γίνεται compile από τον gem5.

Ευελπιστώ να έχω απαντήσει σε όλα τα ερωτήματα της εργασίας όπως ζητήθηκαν κυρίως γιατί στα εκτενεί αρχεία που προέκυπταν από τις προσομοιώσεις δεν γνώριζα αν έπαιρνα σωστά και όλες τις πληροφορίες που έπρεπε να πάρω. Αυτό οφείλεται προφανώς στην περιορισμένη γνώση μου στον συγκεκριμένο αντικείμενο. Τέλος, η εργασία είναι αναμφισβήτητα ενδιαφέρουσα για όσους θέλουν να ασχοληθούν με τον τομέα της ηλεκτρονικής.





