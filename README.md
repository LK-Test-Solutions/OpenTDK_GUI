# OpenTDK_GUI

This repository contains classes to create JavaFX GUI applications and charts.

It allows a quick introduction to the implementation of desktop applications as well as the creation of various diagrams, such as bar charts or line charts.

## Chart Samples

See the test cases of the project.

## Base Application Sample

````java

import java.util.Locale;
import java.util.ResourceBundle;
import java.util.logging.ConsoleHandler;
import java.util.logging.FileHandler;
import java.util.logging.Handler;
import java.util.logging.Level;
import java.util.logging.LogRecord;
import java.util.logging.Logger;
import java.util.logging.SimpleFormatter;

import org.opentdk.gui.application.BaseApplication;
import org.opentdk.gui.controls.MessageDialog.MessageType;

import com.fme.safe.view.RootController;

import javafx.application.Platform;
import javafx.scene.control.ButtonType;
import javafx.scene.image.Image;
import lombok.Getter;
import lombok.Setter;

public final class CustomApp extends BaseApplication {
    /**
     * The one and only instance of this class to get access from other classes without creating another
     * instance there.
     */
    @Getter
    private static CustomApp instance;
    /**
     * The controller class for the main layout.
     */
    @Getter
    private RootController rootController;

    @Getter
    private Logger log;	
    
    /**
     * True: Log file gets written, false: No logging at all.
     */
    @Getter @Setter
    private boolean logEnabled = true;
    /**
     * Initializes the {@link #instance}.
     */
    public CustomApp() {
        if (instance == null) {
            instance = this;
        }
    }
    
    /**
     * Entry point of the <code>CustomApp</code> application where the
     * <code>javafx.Application.launch()</code> method gets called which starts the JavaFX application
     * thread and triggers the {@link #showRootLayout()} method.
     * 
     * @param args unused
     */
    public static void main(String[] args) {
        launch();
    }
    
    /**
     * Close, remove and add log handlers to log to file.
     * Used at startup and to avoid log memory leaks when running for a long time.
     */
    private void initLogger() {
        String logPath = Paths.get("logs" + File.separator + getClass().getSimpleName() + ".log").toString();
        
        log = Logger.getLogger(logPath);
        log.setLevel(Level.ALL);
        log.setUseParentHandlers(false);
        
        for(Handler handler : log.getHandlers()) {
            handler.close();
            log.removeHandler(handler);
        }
        
        if(logEnabled) {
            try {
                ConsoleHandler consoleHandler = new ConsoleHandler();
                consoleHandler.setFormatter(new CustomFormatter());
                log.addHandler(consoleHandler);
    
                FileHandler fileHandler = new FileHandler(log.getName(), true);
                fileHandler.setFormatter(new CustomFormatter());
                log.addHandler(fileHandler);				
            } catch (SecurityException | IOException e) {
                e.printStackTrace();
                log.severe(e.getMessage());				
                Platform.exit();
            }
        }
    }
    
    /**
     * Initialization of the {@link #rootController} that shows the primary stage.
     */
    @Override
    protected void showRootLayout() {
        initLogger();
        super.getPrimaryStage().getIcons().add(new Image("file:conf/app.png"));
        super.setResourceBundle(ResourceBundle.getBundle("test.Bundle", new Locale("en")));
        super.setStyleSheet("app.css");
        super.getPrimaryStage().setResizable(true);
        super.getPrimaryStage().setMinHeight(200);
        super.getPrimaryStage().setMinWidth(200);
        super.getPrimaryStage().setOnCloseRequest(event -> {
            Platform.exit();
        });
        try {
            rootController = (RootController) super.showStage("view/Root.fxml", super.getBundle().getString("app.title"), super.getPrimaryStage());
            rootController.initRootLayout();
        } catch (IOException e) {
            e.printStackTrace();
            log.severe(e.getMessage());				
            Platform.exit();
        } 
    }
    
    public ButtonType showMessageDialog(MessageType type, String info, String message) {
        return getMessageDialog().showMessageBox(type, getBundle().getString(info), getBundle().getString(message));
    }
    
    class CustomFormatter extends SimpleFormatter {
        @Override
        public String format(LogRecord record) {
            return String.format(
                "[%1$tF %1$tT] [%2$s] %3$s: %4$s%n",
                record.getMillis(),
                record.getLevel().getName(),
                record.getLoggerName(),
                record.getMessage()
            );
        }
    }
    }


````

## Run JavaFX in Intellij

![image](https://github.com/user-attachments/assets/357de6b1-8b38-4f5c-9719-c33afeedf9b7)
