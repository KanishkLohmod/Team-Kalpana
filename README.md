import sys
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib.backends.backend_qt5agg import FigureCanvasQTAgg as FigureCanvas
from PyQt5.QtWidgets import (
    QApplication, QMainWindow, QTabWidget, QVBoxLayout, QHBoxLayout,
    QPushButton, QWidget, QLabel, QGridLayout
)
from PyQt5.QtCore import QTimer, Qt
from PyQt5.QtGui import QPixmap


class GraphCanvas(FigureCanvas):
    def __init__(self, parent=None, title=""):
        self.fig, self.ax = plt.subplots()
        super().__init__(self.fig)
        self.setParent(parent)
        self.title = title
        self.ax.set_title(self.title)
    def update_plot(self, x, y):
        self.ax.clear()
        self.ax.plot(x, y, color="blue", linewidth=1.5)
        self.ax.set_title(self.title, fontsize=10)
        self.ax.set_facecolor("#FFFFFF")
        self.ax.grid(color="gray", linestyle="--", linewidth=0.5)
        self.ax.set_xlabel("Time", fontsize=8)
        self.ax.set_ylabel("Value", fontsize=8)
        self.draw()
class MainWindow(QMainWindow):
    def __init__(self, csv_path, logo_path):
        super().__init__()
        self.setWindowTitle("Telemetry Dashboard")
        self.setGeometry(300, 100, 1300, 1000)
        self.setStyleSheet ("background-color: #0b0b0d;")
     # CSV Data
        self.data = pd.read_csv(csv_path)
        self.columns = ["ALTITUDE", "PRESSURE", "VOLTAGE", "GNSS_ALTITUDE", "ACC_R", "GYRO_R"]
        self.simulated_time = []
        self.simulated_data = {col: [] for col in self.columns}
        self.current_index = 0
        # Header Layout
        header_widget = QWidget()
        header_layout = QVBoxLayout(header_widget)
        header_widget.setStyleSheet("background-color: #000066; padding: -230px;")
        # Software State Button
        software_label = QLabel("SOFTWARE STATE")
        software_label.setStyleSheet("font-size: 20px; font-weight: bold; color: #CCFFFF; padding: 5px;")
        header_layout.addWidget(software_label, alignment=Qt.AlignLeft)
        # Launch Pad Button 
        launch_button = QPushButton("LAUNCH_PAD")
        launch_button.setStyleSheet("""
            QPushButton {
                background-color: white; color: #000000;
                font-size: 12px; font-weight: ; padding: 5px; border-radius: 10px;
                border: 2px solid #e74c3c;
            }
            QPushButton:hover  {
                background-color: #c0392b;
            }
        """)
        header_layout.addWidget(launch_button ,1  ,Qt.AlignLeft)
        # Title Label
        title_label = QLabel("TEAM KALPANA: 2024-CANSAT-ASI-023")
        title_label.setStyleSheet("font-size: 25px; font-weight: bold; color: #CCFFFF;")
        title_label.setAlignment(Qt.AlignCenter)
        # Logo 
        logo_label = QLabel()
        pixmap = QPixmap(logo_path).scaled(90,90, Qt.KeepAspectRatio, Qt.SmoothTransformation)
        logo_label.setPixmap(pixmap)
        logo_label.setAlignment(Qt.AlignCenter)
        # Add Widgets to Header Layout
        header_layout.addWidget(title_label)
        header_layout.addWidget(logo_label)
        time_packet_layout = QVBoxLayout()
        self.time_label = QLabel("TIME")
        self.packet_label = QLabel("PACKET COUNT")
        self.time_label.setStyleSheet("""
            QLabel {
                background-color: white; 
                color: black; 
                padding: 10px 15px; 
                border: 2px solid red; 
                border-radius: 10px;
                font-size: 14px;
                font-weight: bold;
               }
        """)
        self.packet_label.setStyleSheet("""
            QLabel {
                background-color: white; 
                color: black; 
                padding: 10px 15px; 
                border: 2px solid red; 
                border-radius: 10px;
                font-size: 14px;
                font-weight: bold;
            }
        """)
        time_packet_layout.addWidget(self.time_label)
        time_packet_layout.addWidget(self.packet_label)
        header_layout.addLayout(time_packet_layout, 10) 
        header_layout.setAlignment(time_packet_layout, Qt.AlignRight)                     
        #self.packet_label = QLabel("PACKET COUNT: 0")
        #self.time_label.setStyleSheet("background-color: #CCFFFF; color: #000000; padding: 5px;")
        #self.packet_label.setStyleSheet("background-color: #CCFFFF; color: #000000; padding: 5px;")
        #time_packet_layout.addWidget(self.time_label,1 ,alignment=Qt.AlignRight)
        #time_packet_layout.addWidget(self.packet_label, ,alignment=Qt.AlignRight)
        #header_layout.addLayout(time_packet_layout)
        self.tabs = QTabWidget()
        self.tabs.setStyleSheet("""
            QTabWidget::pane { background: #ADD8E6; border: 30px solid 	#CC99FF; }
            QTabBar::tab { background: #ADD8E6; color: #000000; padding: 20px; }
            QTabBar::tab:selected { background: #C0C0C0; color: #ffffff; }
        """)
        # Telemetry Data Tab
        self.telemetry_tab = QWidget()
        self.tabs.addTab(self.telemetry_tab, "Telemetry Data")
        # Tab 1: Graphs
        graph_tab = QWidget()
        self.tabs.addTab(graph_tab, "Graphs")
        self.tabs.setStyleSheet("font-weight: bold;background: #cc99ff; color #000000 ; padding: 20px;border-radius: 10px;")
        graph_layout = QGridLayout(graph_tab)
        self.graph_canvases = {}
        for i, col in enumerate(self.columns):
            graph_canvas = GraphCanvas(graph_tab, title=f"{col.capitalize()} vs Time")
            self.graph_canvases[col] = graph_canvas
            graph_layout.addWidget(graph_canvas, i // 3, i % 3)
        # Location and 3D Plotting Tab
        self.location_tab = QWidget()
        self.tabs.addTab(self.location_tab, "Location and 3D Plotting")
        # Live Telecast Tab
        self.live_telecast_tab = QWidget()
        self.tabs.addTab(self.live_telecast_tab, "Live Telecast")
        # Footer Buttons
        footer_layout = QHBoxLayout()
        buttons = ["BOOT", "Set Time", "Calibrate", "ON / OFF", "CX", "SIM Enable", "SIM Activate", "SIM Disable"]
        for button_name in buttons:
            button = QPushButton(button_name)
            button.setStyleSheet("""
               QPushButton {
                     background-color: qlineargradient(
                        spread: pad, x5: 1, y1: 0, x2: 1, y2: 1,
                        stop: 0 #000000, stop: 1 #30BFBF
                    );
                    color: #FFCC00; padding: 15px; border-radius: 5px;
                }
                QPushButton:hover { background-color: #30BFBF; }
            """)
            footer_layout.addWidget(button)
        # Main Layout
        main_layout = QVBoxLayout()
        main_layout.addWidget(header_widget)
        main_layout.addWidget(self.tabs)
        main_layout.addLayout(footer_layout)
        central_widget = QWidget()
        central_widget.setLayout(main_layout)
        self.setCentralWidget(central_widget)
        # Timer
        self.timer = QTimer(self)
        self.timer.timeout.connect(self.update_graphs)
        self.timer.start(1000)
    def update_graphs(self):
        if self.current_index < len(self.data):
            self.simulated_time.append(len(self.simulated_time))
            for col in self.columns:
                self.simulated_data[col].append(self.data[col].iloc[self.current_index])
            for col, canvas in self.graph_canvases.items():
                canvas.update_plot(self.simulated_time, self.simulated_data[col])
            self.current_index += 10
            self.time_label.setText(f"TIME: {self.current_index}")
            self.packet_label.setText(f"PACKET COUNT: {self.current_index}")
if __name__ == "__main__":
    csv_path = "my file.csv"
    logo_path = "logo.png" 
    app = QApplication(sys.argv)
    window = MainWindow(csv_path, logo_path)
    window.show()
    sys.exit(app.exec_())
