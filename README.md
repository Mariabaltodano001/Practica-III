# Practica-III
En este repositorio se realiza la práctica 3 
import Controlador.LoginController;
import Modelo.DataBaseManager;
import Vista.LoginView;
import java.awt.event.ActionListener;

/**
 *
 * @author María José
 */
public class Main {

    public static void main(String[] args) {
        // Inicializar la base de datos
        DataBaseManager databaseManager = new DataBaseManager();

        // Inicializar la vista de inicio de sesión y el controlador
        LoginView loginView = new LoginView();
        LoginController loginController = new LoginController(loginView, databaseManager);
  }
    public void setLoginButtonListener(ActionListener listener) {
        buttonIniciarSesion.addActionListener(listener);
    }
}

import Modelo.Compra;
import Modelo.DataBaseManager;
import Vista.HistorialComprasView;
import java.util.List;

/**
 *
 * @author María José
 */
public class HistorialComprasController {
  private HistorialComprasView historialComprasView;
    private DataBaseManager databaseManager;

    // Constructor
    public HistorialComprasController(HistorialComprasView historialComprasView, DataBaseManager databaseManager) {
        this.historialComprasView = historialComprasView;
        this.databaseManager = databaseManager;
    }

    // Método para cargar y mostrar el historial de compras
    public void mostrarHistorialCompras(int idFuncionario) {
        List<Compra> historialCompras = databaseManager.obtenerHistorialCompras(idFuncionario);
        historialComprasView.mostrarHistorialCompras(historialCompras);
    }
}
package Controlador;
import Modelo.DataBaseManager;
import Vista.LoginView;
import Vista.OrdenView;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import javax.swing.JOptionPane;

/**
 *
 * @author María José
 */
public class LoginController {
    private LoginView loginView;
    private DataBaseManager databaseManager;

    // Constructor
    public LoginController(LoginView loginView, DataBaseManager databaseManager) {
        this.loginView = loginView;
        this.databaseManager = databaseManager;
        this.loginView.setLoginButtonListener(new LoginButtonListener());
    }

    // ActionListener para el botón de inicio de sesión
    class LoginButtonListener implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            String usuario = loginView.getUsuario();
            String contraseña = loginView.getContraseña();

            // Validar las credenciales en la base de datos
            boolean validado = databaseManager.validarCredenciales(usuario, contraseña);

            if (validado) {
                // Si las credenciales son válidas, abrir la vista de órdenes
                // (o cualquier otra acción que deba tomar después de iniciar sesión)
                // Por ejemplo:
                OrdenController ordenController = new OrdenController(new OrdenView(), databaseManager);
            } else {
                // Mostrar mensaje de error
                JOptionPane.showMessageDialog(loginView, "Usuario o contraseña incorrectos", "Error de inicio de sesión", JOptionPane.ERROR_MESSAGE);
            }
        }
    }
}
package Controlador;

import Modelo.DataBaseManager;
import Vista.OrdenView;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

/**
 *
 * @author María José
 */
public class OrdenController {
     private OrdenView ordenView;
    private DataBaseManager databaseManager;

    // Constructor
    public OrdenController(OrdenView ordenView, DataBaseManager databaseManager) {
        this.ordenView = ordenView;
        this.databaseManager = databaseManager;
        this.ordenView.setAgregarProductoButtonListener(new AgregarProductoButtonListener());
        this.ordenView.setRealizarCompraButtonListener(new RealizarCompraButtonListener());
    }

    // ActionListener para el botón de agregar producto
    class AgregarProductoButtonListener implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            // Lógica para agregar un producto a la orden
            // Esto podría implicar actualizar la lista de productos en la vista
            // o agregar el producto seleccionado a una lista en el controlador
        }
    }

    // ActionListener para el botón de realizar compra
    class RealizarCompraButtonListener implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            // Lógica para finalizar la orden y guardarla en la base de datos
            // Esto podría implicar obtener los productos de la orden desde la vista
            // y luego llamar a un método en el DatabaseManager para guardar la orden en la base de datos
        }
    }
    
}
package Modelo;

import java.time.LocalDateTime;
import java.util.List;

/**
 *
 * @author María José
 */
public class Compra {
    private int id;
    private LocalDateTime fechaHora;
    private List<Producto> productos;
    private double total;

    // Constructor
    public Compra(int id, LocalDateTime fechaHora, List<Producto> productos, double total) {
        this.id = id;
        this.fechaHora = fechaHora;
        this.productos = productos;
        this.total = total;
    }

    // Getters y setters
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public LocalDateTime getFechaHora() {
        return fechaHora;
    }

    public void setFechaHora(LocalDateTime fechaHora) {
        this.fechaHora = fechaHora;
    }

    public List<Producto> getProductos() {
        return productos;
    }

    public void setProductos(List<Producto> productos) {
        this.productos = productos;
    }

    public double getTotal() {
        return total;
    }

    public void setTotal(double total) {
        this.total = total;
    }
}
package Modelo;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
/**
 *
 * @author María José
 */
public class DataBaseManager {
 private static final String URL = "jdbc:mysql://localhost:3306/panaderia";
    private static final String USER = "usuario";
    private static final String PASSWORD = "contraseña";

    // Método para obtener una conexión a la base de datos
    private Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }

    // Método para validar las credenciales de un funcionario durante el inicio de sesión
    public boolean validarCredenciales(String usuario, String contraseña) {
        String query = "SELECT * FROM funcionario WHERE usuario = ? AND contraseña = ?";
        try (Connection connection = getConnection();
             PreparedStatement statement = connection.prepareStatement(query)) {
            statement.setString(1, usuario);
            statement.setString(2, contraseña);
            try (ResultSet resultSet = statement.executeQuery()) {
                return resultSet.next(); // Si hay al menos una fila, las credenciales son válidas
            }
        } catch (SQLException e) {
            e.printStackTrace();
            return false;
        }
    }

    // Método para obtener todos los productos de la base de datos
    public List<Producto> obtenerProductos() {
        List<Producto> productos = new ArrayList<>();
        String query = "SELECT * FROM producto";
        try (Connection connection = getConnection();
             PreparedStatement statement = connection.prepareStatement(query);
             ResultSet resultSet = statement.executeQuery()) {
            while (resultSet.next()) {
                int id = resultSet.getInt("id");
                String nombre = resultSet.getString("nombre");
                double precio = resultSet.getDouble("precio");
                productos.add(new Producto(id, nombre, precio));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return productos;
    }

    // Método para guardar una nueva orden de compra en la base de datos
    public void guardarOrdenCompra(int idFuncionario, List<Producto> productos, double total) {
        String queryInsertCompra = "INSERT INTO compra (id_funcionario, fecha_hora, total) VALUES (?, ?, ?)";
        String queryInsertDetalleCompra = "INSERT INTO detalle_compra (id_compra, id_producto) VALUES (?, ?)";
        Connection connection = null;
        try {
            connection = getConnection();
            // Iniciar una transacción
            connection.setAutoCommit(false);

            // Insertar la compra
            LocalDateTime fechaHora = LocalDateTime.now();
            try (PreparedStatement statementInsertCompra = connection.prepareStatement(queryInsertCompra,
                     PreparedStatement.RETURN_GENERATED_KEYS);
                 PreparedStatement statementInsertDetalleCompra = connection.prepareStatement(queryInsertDetalleCompra)) {

                statementInsertCompra.setInt(1, idFuncionario);
                statementInsertCompra.setObject(2, fechaHora);
                statementInsertCompra.setDouble(3, total);
                statementInsertCompra.executeUpdate();

                // Obtener el ID de la compra insertada
                ResultSet generatedKeys = statementInsertCompra.getGeneratedKeys();
                int idCompra = -1;
                if (generatedKeys.next()) {
                    idCompra = generatedKeys.getInt(1);
                }

                // Insertar los detalles de la compra
                for (Producto producto : productos) {
                    statementInsertDetalleCompra.setInt(1, idCompra);
                    statementInsertDetalleCompra.setInt(2, producto.getId());
                    statementInsertDetalleCompra.executeUpdate();
                }

                // Confirmar la transacción
                connection.commit();
            }
        } catch (SQLException e) {
            e.printStackTrace();
            // Si ocurre algún error, hacer rollback de la transacción
            try {
                if (connection != null) {
                    connection.rollback();
                }
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
        } finally {
            // Cerrar la conexión
            try {
                if (connection != null) {
                    connection.close();
                }
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
        }
    }

    // Método para obtener el historial de compras de un funcionario
    public List<Compra> obtenerHistorialCompras(int idFuncionario) {
        List<Compra> historialCompras = new ArrayList<>();
        String query = "SELECT * FROM compra WHERE id_funcionario = ?";
        try (Connection connection = getConnection();
             PreparedStatement statement = connection.prepareStatement(query)) {
            statement.setInt(1, idFuncionario);
            try (ResultSet resultSet = statement.executeQuery()) {
                while (resultSet.next()) {
                    int idCompra = resultSet.getInt("id");
                    LocalDateTime fechaHora = resultSet.getObject("fecha_hora", LocalDateTime.class);
                    double total = resultSet.getDouble("total");
                    // Obtener los productos de la compra
                    List<Producto> productos = obtenerProductosDeCompra(idCompra);
                    historialCompras.add(new Compra(idCompra, fechaHora, productos, total));
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return historialCompras;
    }

    // Método para obtener los productos de una compra específica
    private List<Producto> obtenerProductosDeCompra(int idCompra) {
        List<Producto> productos = new ArrayList<>();
        String query = "SELECT p.* FROM detalle_compra dc JOIN producto p ON dc.id_producto = p.id WHERE dc.id_compra = ?";
        try (Connection connection = getConnection();
             PreparedStatement statement = connection.prepareStatement(query)) {
            statement.setInt(1, idCompra);
            try (ResultSet resultSet = statement.executeQuery()) {
                while (resultSet.next()) {
                    int id = resultSet.getInt("id");
                    String nombre = resultSet.getString("nombre");
                    double precio = resultSet.getDouble("precio");
                    productos.add(new Producto(id, nombre, precio));
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return productos;
    }
}
    
package Modelo;

/**
 *
 * @author María José
 */
public class Funcionario {
    private int id;
    private String nombre;
    private String contraseña;

    // Constructor
    public Funcionario(int id, String nombre, String contraseña) {
        this.id = id;
        this.nombre = nombre;
        this.contraseña = contraseña;
    }

    // Getters y setters
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public String getContraseña() {
        return contraseña;
    }

    public void setContraseña(String contraseña) {
        this.contraseña = contraseña;
    }
} 
    
package Modelo;

/**
 *
 * @author María José
 */
public class Producto {
    private int id;
    private String nombre;
    private double precio;

    // Constructor
    public Producto(int id, String nombre, double precio) {
        this.id = id;
        this.nombre = nombre;
        this.precio = precio;
    }

    // Getters y setters
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public double getPrecio() {
        return precio;
    }

    public void setPrecio(double precio) {
        this.precio = precio;
    }
}
    
package Vista;
import Modelo.Compra;
import Modelo.Producto;
import javax.swing.*;
import java.awt.*;
import java.util.List;
/**
 *
 * @author María José
 */
public class HistorialComprasView extends JFrame {
    private JTextArea textAreaHistorial;

    // Constructor
    public HistorialComprasView(List<Compra> historialCompras) {
        setTitle("Historial de Compras");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(400, 300);

        // Layout
        JPanel panel = new JPanel();
        panel.setLayout(new BorderLayout());

        // Componentes
        textAreaHistorial = new JTextArea();
        JScrollPane scrollPane = new JScrollPane(textAreaHistorial);

        // Mostrar el historial de compras en el área de texto
        mostrarHistorialCompras(historialCompras);

        // Agregar componente al panel
        panel.add(scrollPane, BorderLayout.CENTER);

        // Agregar panel al frame
        add(panel);

        // Mostrar frame
        setLocationRelativeTo(null); // Centrar en la pantalla
        setVisible(true);
    }

    // Método para mostrar el historial de compras en el área de texto
    public void mostrarHistorialCompras(List<Compra> historialCompras) {
        textAreaHistorial.setText(""); // Limpiar el área de texto antes de agregar el historial

        for (Compra compra : historialCompras) {
            textAreaHistorial.append("ID: " + compra.getId() + "\n");
            textAreaHistorial.append("Fecha y Hora: " + compra.getFechaHora() + "\n");

            // Mostrar productos de la compra
            textAreaHistorial.append("Productos:\n");
            for (Producto producto : compra.getProductos()) {
                textAreaHistorial.append("- " + producto.getNombre() + " - $" + producto.getPrecio() + "\n");
            }

            textAreaHistorial.append("Total: $" + compra.getTotal() + "\n\n");
        }
    }
}
package Vista;
import Controlador.LoginController;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
/**
 *
 * @author María José
 */
public class LoginView extends JFrame {
       private JTextField textFieldUsuario;
    private JPasswordField passwordFieldContraseña;
    private JButton buttonIniciarSesion;   

    // Constructor
    public LoginView() {
        setTitle("Inicio de Sesión");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(300, 150);

        // Layout
        JPanel panel = new JPanel();
        panel.setLayout(new GridLayout(3, 2));

        // Componentes
        JLabel labelUsuario = new JLabel("Usuario:");
        textFieldUsuario = new JTextField();
        JLabel labelContraseña = new JLabel("Contraseña:");
        passwordFieldContraseña = new JPasswordField();
        buttonIniciarSesion = new JButton("Iniciar Sesión");

        // Acción del botón de inicio de sesión
        buttonIniciarSesion.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                // Lógica para iniciar sesión
                String usuario = textFieldUsuario.getText();
                String contraseña = new String(passwordFieldContraseña.getPassword());
                // Llamar al controlador para validar el inicio de sesión
            }
        });

        // Agregar componentes al panel
        panel.add(labelUsuario);
        panel.add(textFieldUsuario);
        panel.add(labelContraseña);
        panel.add(passwordFieldContraseña);
        panel.add(new JLabel()); // Espacio en blanco para alinear los componentes
        panel.add(buttonIniciarSesion);

        // Agregar panel al frame
        add(panel);

        // Mostrar frame
        setLocationRelativeTo(null); // Centrar en la pantalla
        setVisible(true);
    }

    // Método para limpiar los campos de texto después del inicio de sesión
    public void limpiarCampos() {
        textFieldUsuario.setText("");
        passwordFieldContraseña.setText("");
    }

    // Método para establecer el ActionListener para el botón de inicio de sesión
    public void setLoginButtonListener(ActionListener listener) {
        buttonIniciarSesion.addActionListener(listener);
    }

    // Método para obtener el nombre de usuario ingresado
    public String getUsuario() {
        return textFieldUsuario.getText();
    }

    // Método para obtener la contraseña ingresada
    public String getContraseña() {
        return new String(passwordFieldContraseña.getPassword());
    }
}
package Vista;
import Controlador.OrdenController;
import Modelo.Producto;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.List;
/**
 *
 * @author María José
 */

public class OrdenView extends JFrame {
   private JComboBox<Producto> comboBoxProductos;
    private JButton buttonAgregarProducto;
    private JTextArea textAreaOrden;
    private JButton buttonRealizarCompra;

    // Constructor
    OrdenView(List<Producto> listaProductos) {
        setTitle("Realizar Orden");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(400, 300);

        // Layout
        JPanel panel = new JPanel();
        panel.setLayout(new BorderLayout());

        // Componentes
        comboBoxProductos = new JComboBox<>(listaProductos.toArray(new Producto[0]));
        buttonAgregarProducto = new JButton("Agregar Producto");
        textAreaOrden = new JTextArea();
        buttonRealizarCompra = new JButton("Realizar Compra");

        // Acción del botón de agregar producto
        buttonAgregarProducto.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                Producto productoSeleccionado = (Producto) comboBoxProductos.getSelectedItem();
                textAreaOrden.append(productoSeleccionado.getNombre() + " - $" + productoSeleccionado.getPrecio() + "\n");
            }
        });

        // Acción del botón de realizar compra
        buttonRealizarCompra.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                // Lógica para realizar la compra
            }
        });

        // Panel superior para seleccionar productos
        JPanel panelSuperior = new JPanel(new FlowLayout());
        panelSuperior.add(comboBoxProductos);
        panelSuperior.add(buttonAgregarProducto);

        // Panel central para mostrar la orden
        JScrollPane scrollPane = new JScrollPane(textAreaOrden);
        textAreaOrden.setEditable(false);
        textAreaOrden.setRows(10);

        // Panel inferior para el botón de realizar compra
        JPanel panelInferior = new JPanel(new FlowLayout());
        panelInferior.add(buttonRealizarCompra);

        // Agregar componentes al panel principal
        panel.add(panelSuperior, BorderLayout.NORTH);
        panel.add(scrollPane, BorderLayout.CENTER);
        panel.add(panelInferior, BorderLayout.SOUTH);

        // Agregar panel al frame
        add(panel);

        // Mostrar frame
        setLocationRelativeTo(null); // Centrar en la pantalla
        setVisible(true);
    }

    public OrdenView() {
        throw new UnsupportedOperationException("Not supported yet."); // Generated from nbfs://nbhost/SystemFileSystem/Templates/Classes/Code/GeneratedMethodBody
    }

    // Método para actualizar la lista de productos disponibles
    public void actualizarListaProductos(List<Producto> listaProductos) {
        comboBoxProductos.removeAllItems();
        for (Producto producto : listaProductos) {
            comboBoxProductos.addItem(producto);
        }
    }

    // Método para limpiar el área de la orden después de realizar la compra
    public void limpiarOrden() {
        textAreaOrden.setText("");
    }

    // Método para establecer el ActionListener para el botón de agregar producto
    public void setAgregarProductoButtonListener(ActionListener listener) {
        buttonAgregarProducto.addActionListener(listener);
    }

    // Método para establecer el ActionListener para el botón de realizar compra
    public void setRealizarCompraButtonListener(ActionListener listener) {
        buttonRealizarCompra.addActionListener(listener);
    }
}
