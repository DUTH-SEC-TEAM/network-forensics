### **Περίληψη** 

Το συγκεκριμένο lab χρησιμοποιείται για να γίνει κατανοητή η επικοινωνία μεταξύ δύο virtual machines, ενός attacker και ενός victim, καθώς και η διαδικασία με την οποία εντοπίζουμε μια συγκεκριμένη επικοινωνία στην εφαρμογή του Wireshark, για να διαβάσουμε credentials για την σύνδεση σε μια ιστοσελίδα.

---
### **Victim Machine (Debian)**

### **Βήμα 1ο - Εγκατάσταση βιβλιοθήκης sudo** 

1. Ανοίγουμε το Debian Virtual Machine, βάζοντας τα credentials μας.
2. Μπαίνουμε σε περιβάλλον root με την εντολή `su -`.
3. Κατεβάζουμε το sudo package με `apt install sudo`.
4. Βάζουμε τον εαυτό μας στο group των sudoers `usermod -aG sudo TOUSERNAMESOY`.
5. Γράφουμε δύο φορές `exit` για να γίνουν apply οι αλλαγές.
6. Κάνουμε ενημέρωση του συστήματος με `apt update && apt upgrade`.
7. Πατάμε `sudo su` , αν μας ζητήσει κωδικό βάζουμε το δικό μας.
--- 
### **Βήμα 2ο - Εγκατάσταση υπηρεσίας apache και php dependencies**

1. Τρέχουμε `set -a` σε root κατάσταση. (χρησιμοποιείται για να οριστικοποιήσουμε τις μεταβλητές μας, όπως η εντολή `export` ).
2. Κάνουμε ενημέρωση εκτελώντας `apt update`.
3. Κατεβάζουμε την υπηρεσία `apache` και τα `php dependencies` εκτελώντας `apt install -y apache2 php libapache2-mod-php`.
4. Μπαίνουμε μέσα στο φάκελο /var/www/html `cd /var/www/html`.
---
**!!! Πριν να μπούμε στο φάκελο του /var/www/html καλό είναι να κλείσουμε το VM και να αλλάξουμε από τις ρυθμίσεις την σύνδεση του VM, με δεξί κλικ πάνω του και `Settings -> Network -> Attached to: Bridged Adapter` . Άμα ανοίξατε είδη τον φάκελο, δεν πειράζει μόλις ανοίξετε ξανά το VM μπορείτε να ξαναμπείτε, πάντα όμως με κατάσταση root.**

---
### **Βήμα 3ο - Φόρτωση του login page** 

1. Στα Windows μας συνδεόμαστε μέσω command prompt με ssh στο Debian Machine μας `ssh username@ip_debian`.
2. Μπαίνουμε σε κατάσταση root και στο φάκελο /var/www/html 
   ```
   sudo su
   cd /var/www/html
   ```

3. Δημιουργούμε αρχείο για το login page μας `nano login.php` 
4. Κάνουμε επικόληση τον κώδικα στο [[Κώδικας για Login Page]].
5. Πατάμε `Ctrl+ O -> Enter -> Ctlr+ X` για αποθήκευση.
6. Σβήνουμε το αρχείο index.php εκτελώντας `rm -f index.php`.
7. Eνεργοποιούμε τον apache server `systemctl enable apache2`.
8. Κάνουμε restart τον apache `systemctl restart apache2`.
---
### **Attacker Machine (Kali)**

Έχοντας την IP του Victim Machine `ip a` (κάτω από το enp0s3) πρέπει να ακολουθήσουμε τις εξής ενέργειες:

1. Κάνουμε ping τον victim να δούμε αν απαντάει `ping IP`.
2. Κάνουμε nmap scan για να βρούμε τι πόρτες έχουμε ανοιχτές, παίρνοντας παράλληλα πληροφορίες για την IP που έχουμε `nmap -sC -sV IP`.
3. Βλέπουμε από το nmap ότι πρόκειται για web υπηρεσία, οπότε βάζουμε την IP στον browser μας.
4. Ανοίγουμε το wireshark και επιλέγουμε από τα captures `eth0`.
5. Πατάμε το link για το login page.
6. Προσπαθούμε να βάλουμε τα σωστά credentials (μπορούμε να χρησιμοποιήσουμε και το εργαλείο hydra παίρνοντας POST request από το response του server με λάθος credentials).
7. Βάζουμε τα σωστά credentials (user: admin , password: administrator) και κοιτάμε στο Wireshark την απάντηση του server κατά το Login μας (στήλες IP, πρωτόκολλο TCP, SYN-ACK,FIN).
8. Κάνουμε δεξί κλικ στην επικοινωνία `Follow -> TCP stream`.
9. Βλέπουμε το αποτέλεσμα της επικοινωνίας. 
---









