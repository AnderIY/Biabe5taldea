package GUI;

import java.awt.BorderLayout;
import java.awt.EventQueue;
import java.awt.Graphics;
import java.awt.GridLayout;
import java.awt.Toolkit;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

import javax.swing.ImageIcon;
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JOptionPane;
import javax.swing.JPanel;
import javax.swing.JPasswordField;
import javax.swing.JTextField;
import javax.swing.SwingConstants;

import java.sql.*;
import java.util.Arrays;
import java.util.List;
import java.awt.Color;


public class LoginForm {

    private JFrame frame;
    private JTextField textFieldUsuario;
    private JPasswordField passwordFieldContrasena;
    
    public static void main(String[] args) {
        EventQueue.invokeLater(new Runnable() {
            public void run() {
                try {

                    LoginForm window = new LoginForm();
                    window.frame.setVisible(true);
                    
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
    }
    private boolean tienePermisosEditar(String usuario) {
     
        List<String> usuariosConPermisosEditar = Arrays.asList("Ander", "Murua", "Iturrioz");
        
        
        return usuariosConPermisosEditar.contains(usuario);
    }
    public LoginForm() {
        initialize();
    }

    private void initialize() {
        frame = new JFrame("Sesio hasiera");
        frame.getContentPane().setForeground(new Color(21, 31, 66));
        frame.setBackground(new Color(0, 0, 0));
        frame.getContentPane().setBackground(new Color(21, 31, 66));
        frame.setBounds(100, 100, 403, 193);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.getContentPane().setLayout(null);  
        String rutaRelativa = "/GUI/BIABE.jpeg";
        java.net.URL url = getClass().getResource(rutaRelativa);
        ImageIcon Biabe = new ImageIcon(url);
        
        JLabel label1 = new JLabel(Biabe);
        label1.setBounds(10, 91, 111, 52);
        frame.getContentPane().add(label1);
      
       
         
        JLabel lblUsuario = new JLabel("Usuario:");
        lblUsuario.setForeground(new Color(255, 255, 255));
        lblUsuario.setBounds(30, 30, 80, 20);
        frame.getContentPane().add(lblUsuario);

        
        
        textFieldUsuario = new JTextField();
        textFieldUsuario.setBounds(120, 30, 200, 20);
        frame.getContentPane().add(textFieldUsuario);
        textFieldUsuario.setColumns(10);

        JLabel lblContrasena = new JLabel("Pasahitza:");
        lblContrasena.setForeground(new Color(255, 255, 255));
        lblContrasena.setBackground(new Color(0, 0, 0));
        lblContrasena.setBounds(30, 60, 80, 20);
        frame.getContentPane().add(lblContrasena);

        passwordFieldContrasena = new JPasswordField();
        passwordFieldContrasena.setBounds(120, 61, 200, 20);
        frame.getContentPane().add(passwordFieldContrasena);

        JButton btnIniciarSesion = new JButton("Sesioa Hasi");
        btnIniciarSesion.setBounds(151, 102, 100, 30);
        frame.getContentPane().add(btnIniciarSesion);
        
     
    
        
      
        
   
        btnIniciarSesion.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String usuario = textFieldUsuario.getText();
                String contrasena = new String(passwordFieldContrasena.getPassword());

                if (verificarCredenciales(usuario, contrasena)) {
                    frame.dispose();
                    Taulak tabla = new Taulak();
                    if (!tienePermisosEditar(usuario)) {
                        tabla.deshabilitarEdicion(); 
                    }
                    tabla.setVisible(true);
                } else {
                    JOptionPane.showMessageDialog(frame, "Pasahitza edo Usuarioa gaizki sartu duzu", "Error", JOptionPane.ERROR_MESSAGE);
                }
            }
        });
    }

    private boolean verificarCredenciales(String usuario, String contrasena) {
        String jdbcUrl = "jdbc:mysql://192.168.5.1:3306/biabe5taldea";
        String dbUsuario = "root";
        String dbContrasena = "";

        try (Connection conexion = DriverManager.getConnection(jdbcUrl, dbUsuario, dbContrasena)) {
            String consulta = "SELECT * FROM usuarios WHERE usuario = ? AND contraseña = ?";
            PreparedStatement preparedStatement = conexion.prepareStatement(consulta);
            preparedStatement.setString(1, usuario);
            preparedStatement.setString(2, contrasena);
            ResultSet resultado = preparedStatement.executeQuery();

            return resultado.next();
        } catch (SQLException e) {
            e.printStackTrace();
            return false;
        }
    }
}
