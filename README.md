**Code Review Bericht**

**Projekt:** The American Dream Autoverleih

**Reviewer:** [Ihr Name]

**Datum:** [Heutiges Datum]

---

## 1. Allgemeine Anmerkungen
Das Projekt ist ein Java-basiertes Autovermietungssystem, das mit JavaFX als GUI-Technologie implementiert wurde. Es verwendet JSON zur Speicherung von Benutzerdaten und setzt Maven als Build-Tool ein. Die Codequalität ist insgesamt solide, aber es gibt einige Clean-Code-Verstöße und Verbesserungspotenziale, die behoben werden sollten.

---

## 2. Identifizierte Probleme und Verbesserungsvorschläge

### **2.1 App.java**
**Probleme:**
- **Globale Variable `scene`**: Die `scene`-Variable ist statisch, was zu unerwarteten Nebenwirkungen führen kann.
- **Fehlende Fehlerbehandlung**: `loadFXML()` wirft eine `IOException`, aber es gibt keine Logging- oder Fehlerbehandlung.
- **Fehlende Javadoc-Kommentare**: Methoden sollten dokumentiert werden.
- **`setRoot()` ist package-private**: Sollte `private` sein, falls sie nicht von außen benötigt wird.

**Verbesserung:**
```java
private Scene scene;

@Override
public void start(Stage stage) {
    try {
        scene = new Scene(loadFXML("login"));
        stage.setScene(scene);
        stage.show();
    } catch (IOException e) {
        System.err.println("Fehler beim Laden des FXML: " + e.getMessage());
    }
}
```

---

### **2.2 User.java**
**Probleme:**
- **Passwort wird in `toString()` ausgegeben**, was ein Sicherheitsrisiko darstellt.
- **Fehlende Setter und Immutability**: Falls das Passwort nicht verändert werden soll, sollte es `final` sein.
- **Nutzung eines Java `record` möglich** (ab Java 14).

**Verbesserung:**
```java
public record User(String name, String email) {}
```
Falls ein Setter benötigt wird:
```java
public class User {
    private final String name;
    private final String email;
    private String password;

    public User(String name, String email, String password) {
        this.name = name;
        this.email = email;
        this.password = password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

---

### **2.3 JsonUtil.java**
**Probleme:**
- **Manuelles Parsen von JSON ist fehleranfällig** (z. B. Nutzung von `split("(?<=}),")`).
- **Fehlende Fehlerbehandlung und Logging**.
- **Dateipfad (`FILE_PATH`) ist fest im Code verdrahtet**.
- **Ineffiziente String-Konvertierung**.

**Verbesserung mit Gson:**
```java
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;

public class JsonUtil {
    private static final String FILE_PATH = "users.json";

    public static List<User> readUsers() {
        try {
            String content = Files.readString(Paths.get(FILE_PATH));
            return new Gson().fromJson(content, new TypeToken<List<User>>() {}.getType());
        } catch (IOException e) {
            System.err.println("Fehler beim Lesen der Datei: " + e.getMessage());
            return List.of();
        }
    }
}
```

---

## 3. Fazit
- **Wichtige Verbesserungen**:
  - Nutzung einer JSON-Bibliothek anstelle manueller String-Manipulation.
  - Entfernen von sicherheitskritischen Fehlern (Passwort in `toString()`).
  - Logging und Exception-Handling verbessern.
  - Methoden besser dokumentieren und die Sichtbarkeit von Variablen und Methoden optimieren.

Durch diese Verbesserungen wird der Code robuster, sicherer und leichter wartbar.

