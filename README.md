# Java Project: Password Manager

## Objective

Develop a secure, user-friendly desktop application for managing passwords. The system allows users to register, log in, and manage (add, edit, delete, import, export) their passwords for various sites. The application is built with a strong focus on Java Swing for GUI, robust validation, and persistent storage using SQLite.

---

## Technology Stack

- **GUI:** Java Swing (extensive use of JFrame, JPanel, JTable, dialogs, menu bars, custom feedback)
- **Database:** SQLite (via JDBC)
- **Event Handling:** ActionListener, KeyListener, MouseListener, ItemListener
- **OOP:** Modular, MVC-inspired structure

---

## Application Structure & Swing Usage

### 1. Main Entry Point

```java
public class Main {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(AuthPage::new);
    }
}
```
- Launches the application on the Swing event dispatch thread, ensuring thread safety for all UI operations.

---

### 2. Authentication (Login & Registration)

#### AuthPage (JFrame)
- The main window for authentication, containing an `AuthPanel`.
- Switches to the dashboard upon successful login.

#### AuthPanel (JPanel)
- Uses a `CardLayout` to switch between login and registration forms.
- **Login Panel:**
  - Username and password fields (`JTextField`, `JPasswordField`)
  - Show/hide password (`JCheckBox`)
  - Forgot password button (shows password hint via dialog)
  - Keyboard navigation (Enter to submit, Alt+R/L to switch forms, Esc to clear fields)
- **Register Panel:**
  - Username, password, and password hint fields
  - Show/hide password, generate password button
  - **Dynamic requirements panel:**
    - Shows real-time feedback on username and password requirements (length, character types, etc.)
    - Uses Unicode checkmarks/crosses for clarity
  - Register button is only enabled when all requirements are met
  - Keyboard navigation (Enter to submit, Alt+L to switch, Esc to clear)

**Example: Dynamic Requirements Feedback**
```java
requirements.setText(
    (hasLength ? "✓" : "✗") + " At least 8 characters\n" +
    (hasUpper ? "✓" : "✗") + " At least one uppercase letter\n" +
    (hasLower ? "✓" : "✗") + " At least one lowercase letter\n" +
    (hasNumber ? "✓" : "✗") + " At least one number\n" +
    (hasSpecial ? "✓" : "✗") + " At least one special character"
);
```

**Example: Show/Hide Password**
```java
showPasswordLogin.addActionListener(_ ->
    loginPassField.setEchoChar(showPasswordLogin.isSelected() ? '\0' : '\u2022')
);
```

**Example: CardLayout for Switching Forms**
```java
CardLayout cardLayout = new CardLayout();
setLayout(cardLayout);
add(loginPanel, "LOGIN");
add(registerPanel, "REGISTER");
cardLayout.show(this, "LOGIN");
```

**Example: Keyboard Navigation**
```java
KeyStroke altR = KeyStroke.getKeyStroke(KeyEvent.VK_R, InputEvent.ALT_DOWN_MASK);
loginPanel.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW).put(altR, "switchToRegister");
loginPanel.getActionMap().put("switchToRegister", new AbstractAction() {
    public void actionPerformed(ActionEvent e) {
        cardLayout.show(AuthPanel.this, "REGISTER");
    }
});
```

---

### 3. Dashboard (Main Application Window)

#### DashboardPage (JFrame)
- Contains a `DashboardPanel` and a custom menu bar.
- **Menu Bar Features:**
  - File: Import/Export passwords (JSON), Logout, Exit
  - View: Toggle Dark Mode (switches all component colors recursively)
  - Help: About dialog (shows app info)
  - Profile: View/Edit profile (change password/hint with strength check), Delete profile
- **Dark Mode:**
  - Recursively updates background/foreground colors for all components
  - Also updates JTable and header colors

**Example: Menu Bar Setup**
```java
JMenuBar menuBar = new JMenuBar();
JMenu fileMenu = new JMenu("File");
JMenuItem importItem = new JMenuItem("Import");
importItem.addActionListener(_ -> handleImport());
fileMenu.add(importItem);
menuBar.add(fileMenu);
setJMenuBar(menuBar);
```

**Example: Dark Mode Toggle**
```java
private void setDarkMode() {
    Color bgColor = new Color(43, 43, 43);
    Color fgColor = new Color(200, 200, 200);
    setComponentColors(this, bgColor, fgColor);
    dashboardPanel.getPasswordTable().setBackground(bgColor);
    ...
}
```

**Example: About Dialog**
```java
JOptionPane.showMessageDialog(this,
    "Password Manager by Asmin\nA secure password management application\n\u00a9 2025 Password Manager",
    "About Password Manager",
    JOptionPane.INFORMATION_MESSAGE);
```

---

### 4. Password Management Panel (DashboardPanel)

#### Layout & Components
- **Top:** Search bar, filter combo box (All/Site/Username), clear search
- **Center:** JTable for passwords (site, username, password)
- **Bottom:** Add, Edit, Delete, Show/Hide Passwords, Logout buttons

**Example: JTable Setup**
```java
String[] columns = {"Site", "Username", "Password"};
tableModel = new DefaultTableModel(columns, 0) {
    public boolean isCellEditable(int row, int column) { return false; }
};
passwordTable = new JTable(tableModel);
passwordTable.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
```

**Example: Context Menu for JTable**
```java
JPopupMenu contextMenu = new JPopupMenu();
JMenuItem copyUsernameItem = new JMenuItem("Copy Username");
copyUsernameItem.addActionListener(_ -> copyToClipboard(username));
contextMenu.add(copyUsernameItem);
passwordTable.setComponentPopupMenu(contextMenu);
```

**Example: Show/Hide Password Column**
```java
TableColumn passwordColumn = passwordTable.getColumnModel().getColumn(2);
passwordColumn.setMinWidth(0);
passwordColumn.setMaxWidth(0);
passwordColumn.setWidth(0);
passwordColumn.setPreferredWidth(0);
```

**Example: Add/Edit Password Dialog**
```java
JDialog dialog = new JDialog((Frame) SwingUtilities.getWindowAncestor(this), "Add New Password", true);
dialog.setLayout(new GridBagLayout());
JTextField siteField = new JTextField(20);
JPasswordField passwordField = new JPasswordField(20);
JCheckBox showPassword = new JCheckBox("Show Password");
JButton generateButton = new JButton("Generate Password");
JLabel strengthLabel = new JLabel("Password Strength: ");
// ... add components and listeners
```

**Example: Keyboard Shortcuts**
```java
KeyStroke ctrlF = KeyStroke.getKeyStroke(KeyEvent.VK_F, InputEvent.CTRL_DOWN_MASK);
getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW).put(ctrlF, "focusSearch");
getActionMap().put("focusSearch", new AbstractAction() {
    public void actionPerformed(ActionEvent e) {
        searchField.requestFocus();
    }
});
```

**Example: Search & Filter**
```java
filterComboBox.addActionListener(_ -> performSearch());
searchField.addKeyListener(new KeyAdapter() {
    public void keyPressed(KeyEvent e) {
        if (e.getKeyCode() == KeyEvent.VK_ENTER) performSearch();
    }
});
```

**Example: Import/Export**
```java
// Export passwords to JSON
JSONArray jsonArray = new JSONArray();
for (Password pwd : passwords) {
    JSONObject entry = new JSONObject();
    entry.put("site", pwd.getSite());
    entry.put("username", pwd.getUsername());
    entry.put("password", pwd.getPassword());
    jsonArray.put(entry);
}
Files.write(Paths.get(filePath), jsonArray.toString(2).getBytes());
```

---

### 5. Profile Management
- Accessible from the menu bar
- View/edit username (read-only), password hint, and password
- Password field with show/hide, generate, and real-time strength check
- Only allows saving if password is "Strong" (enforced by color-coded feedback)
- Delete profile: Confirms, deletes user, returns to login

**Example: Profile Dialog**
```java
JTextField usernameField = new JTextField(user.getUsername());
JTextField hintField = new JTextField(user.getPasswordHint());
JPasswordField passwordField = new JPasswordField();
JCheckBox showPasswordCheckBox = new JCheckBox("Show Password");
JButton generatePasswordButton = new JButton("Generate Password");
JLabel strengthLabel = new JLabel("Password Strength: ");
// ... add listeners and layout
```

---

### 6. Password Strength & Generation (Utils)

#### PasswordStrengthChecker
- Evaluates password and returns a message and color (Weak, Fair, Good, Strong)
- Used in registration, add/edit dialogs, and profile management

**Example: Password Strength Check**
```java
PasswordStrengthChecker.PasswordStrength strength = PasswordStrengthChecker.checkStrength(password);
strengthLabel.setText("Password Strength: " + strength.message());
strengthLabel.setForeground(strength.color());
```

#### PasswordGenerator
- Generates a random password with at least one lowercase, uppercase, number, and special character
- Used in registration, add/edit dialogs, and profile management

**Example: Password Generation**
```java
String generatedPassword = PasswordGenerator.generatePassword(16);
passwordField.setText(generatedPassword);
showPassword.setSelected(true);
passwordField.setEchoChar('\0');
```

---

### 7. Database Helper (DBHelper)
- Handles all SQLite operations (users, passwords)
- Uses prepared statements for security
- Manages user registration, authentication, password CRUD, profile updates, and import/export

**Example: Register User**
```java
public static boolean registerUser(String username, String password, String passwordHint) {
    try (Connection conn = DriverManager.getConnection(DB_URL);
         PreparedStatement ps = conn.prepareStatement(
                 "INSERT INTO users (username, password, password_hint) VALUES (?, ?, ?)")) {
        ps.setString(1, username);
        ps.setString(2, password);
        ps.setString(3, passwordHint);
        ps.executeUpdate();
        return true;
    } catch (SQLException e) {
        return false;
    }
}
```

**Example: Authenticate User**
```java
public static User authenticateUser(String username, String password) {
    try (Connection conn = DriverManager.getConnection(DB_URL);
         PreparedStatement ps = conn.prepareStatement(
                 "SELECT * FROM users WHERE username=? AND password=?")) {
        ps.setString(1, username);
        ps.setString(2, password);
        ResultSet rs = ps.executeQuery();
        if (rs.next()) {
            return new User(
                    rs.getInt("id"),
                    rs.getString("username"),
                    rs.getString("password"),
                    rs.getString("password_hint")
            );
        }
    } catch (SQLException e) {
        // handle error
    }
    return null;
}
```

---

## Example UI Flows (Swing Focus)

### Registration Flow
1. User enters username → requirements panel updates in real time
2. User enters password → requirements panel switches to password rules, updates in real time
3. Register button only enabled when all requirements are met
4. On success, dialog shows confirmation

### Add Password Flow
1. User clicks Add → modal dialog opens
2. User enters site, username, password
3. Password field shows real-time strength, can generate password
4. Save adds to table and database, dialog closes

### Menu Bar & Dark Mode
1. User toggles dark mode → all UI colors update instantly
2. User can import/export passwords via dialogs
3. Profile management and about dialogs are accessible from the menu

---

## Modularity & OOP
- Controllers: `AuthController`, `PasswordController` (business logic)
- Models: `User`, `Password` (data)
- Utils: `DBHelper`, `PasswordGenerator`, `PasswordStrengthChecker`
- Views: `AuthPanel`, `DashboardPanel` (UI components)
- Pages: `AuthPage`, `DashboardPage` (main windows)

---

## Conclusion

This Password Manager project demonstrates advanced, practical use of Java Swing for a modern, secure, and user-friendly application. It features:
- Dynamic, real-time feedback for user actions
- Rich dialogs and menu bars
- Keyboard accessibility and shortcuts
- Modular, maintainable code
- Robust validation and security features

**The project is a comprehensive demonstration of Swing's capabilities for desktop application development.**
