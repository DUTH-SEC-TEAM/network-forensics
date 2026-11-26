```
<?php 
// Σκόπιμα ευάλωτη σελίδα login για προσωπικό εργαστήριο ασφάλειας (lab). 
// Hardcoded credentials, καθόλου rate limiting, καθόλου lockout, καθόλου CAPTCHA. 
// ΜΗΝ το εκθέσεις ποτέ εκτός απομονωμένου lab δικτύου. 
$valid_user = "admin"; 
$valid_pass = "administrator"; 
if ($_SERVER["REQUEST_METHOD"] === "POST") {
	$user = isset($_POST["username"]) ? $_POST["username"] : ""; 
	$pass = isset($_POST["password"]) ? $_POST["password"] : ""; 
if ($user === $valid_user && $pass === $valid_pass) { 
// Επιτυχές login: καθαρό 200, χωρίς ιδιαίτερο μήνυμα στο περιεχόμενο. 
    http_response_code(200); 
	echo "<!DOCTYPE html><html><body><h1>Welcome, admin.</h1></body></html>"; 
} else { 
// Αποτυχημένο login: διαφορετικός status code ώστε να ξεχωρίζει εύκολα στο Wireshark. 
	http_response_code(401); 
	echo "<!DOCTYPE html><html><body><h1>Login failed.</h1></body></html>"; 
} 
exit;
} 
?> 
<!DOCTYPE html> 
<html> 
<head><title>Admin Login</title></head> 
<body> 
<h2>Admin Panel</h2> 
<form method="POST" action="">
	<label>Username: <input type="text" name="username" autocomplete="off"></label><br><br> 
	<label>Password: <input type="password" name="password" autocomplete="off"></label><br><br> 
	<input type="submit" value="Login"> 
</form> 
</body>
</html>
```

