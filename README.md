# coding: utf-8
"""
Slim HeaderBar for PyQt5 + QFluentWidgets
──────────────────────────────────────────
• Alarm bell(배지)   • KST / UTC 시계 + 경과 타이머
• Ping / DB 상태    • CPU·RAM 미니 게이지
• E-STOP • Avatar 드롭다운 • Search 박스
"""

import os, sys, psutil, random, time
from datetime import datetime
from PyQt5.QtCore    import Qt, QTimer
from PyQt5.QtGui     import QColor, QPainter, QPen, QFont
from PyQt5.QtWidgets import (
    QWidget, QLabel, QHBoxLayout, QVBoxLayout,
    QProgressBar, QAction
)
from qfluentwidgets import (
    ToolButton, PrimaryToolButton, AvatarWidget,
    RoundMenu, SearchLineEdit, InfoBar,
    FluentIcon as FI, Theme, setTheme
)

# ──────────────────────────
# 🔸  LED 점 (빨·초·주·회)
# ──────────────────────────
class LED(QWidget):
    COLOR = {"red": "#d9534f", "green": "#5cb85c",
             "orange": "#f0ad4e", "grey": "#9e9e9e"}
    def __init__(self, color="grey", parent=None):
        super().__init__(parent)
        self._c = color
        self.setFixedSize(10, 10)
    def setColor(self, c): self._c = c; self.update()
    def paintEvent(self, _):
        qp = QPainter(self)
        qp.setRenderHint(QPainter.Antialiasing)
        qp.setPen(Qt.NoPen)
        qp.setBrush(QColor(self.COLOR[self._c]))
        r = self.width()/2 - 1
        qp.drawEllipse(self.rect().center(), r, r)

# ──────────────────────────
# 🔸  Badge 기능을 갖춘 ToolButton
# ──────────────────────────
class BadgedToolButton(QWidget):
    def __init__(self, icon=FI.BELL, badge=0, parent=None):
        super().__init__(parent)
        self.btn   = ToolButton(icon, self)
        self.btn.setFixedSize(36, 36)
        self.btn.setIconSize(self.btn.size())
        self.badge = QLabel(str(badge), self)
        self.badge.setAlignment(Qt.AlignCenter)
        self.badge.setFont(QFont("Segoe UI", 8))
        self.badge.setStyleSheet("""
            QLabel{
              background:#d9534f; color:white;
              border-radius:7px;
              min-width:14px; min-height:14px;}
        """)
        self.badge.move(self.btn.width()-12, 2)
        self.setBadge(badge)
        self.setFixedSize(self.btn.size())
    def setBadge(self, n:int):
        self.badge.setText(str(n))
        self.badge.setVisible(n>0)
    def setToolTip(self, t): self.btn.setToolTip(t)
    def clicked(self, func): self.btn.clicked.connect(func)

# ──────────────────────────
# 🔸  HeaderBar
# ──────────────────────────
class HeaderBar(QWidget):
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setFixedHeight(48)
        self.setObjectName("HeaderBar")
        self._load_qss()

        h = QHBoxLayout(self); h.setContentsMargins(12, 4, 12, 4); h.setSpacing(16)

        # 1) Alarm
        self.alarm = BadgedToolButton(FI.BELL, 0, self)
        self.alarm.setToolTip("Alarms")
        self.alarm.clicked(self._showAlarms)
        h.addWidget(self.alarm)

        # 2) KST / UTC 시계 + 타이머
        self.kst=QLabel("--:--:-- KST"); self.utc=QLabel("--:--:-- UTC")
        self.timer=QLabel("⏱  --:--")
        vClock=QVBoxLayout(); vClock.setSpacing(0)
        vClock.addWidget(self.kst); vClock.addWidget(self.utc)
        h.addLayout(vClock); h.addWidget(self.timer)

        # 3) Ping / DB 상태
        self.netLed, self.netLbl = LED(), QLabel("Ping -- ms")
        self.dbLed , self.dbLbl  = LED(),  QLabel("DB N/A")
        for lbl in (self.netLbl,self.dbLbl): lbl.setStyleSheet("font-size:11px;")
        netLine=QHBoxLayout(); netLine.addWidget(self.netLed); netLine.addWidget(self.netLbl)
        dbLine =QHBoxLayout(); dbLine .addWidget(self.dbLed ); dbLine .addWidget(self.dbLbl )
        vNet=QVBoxLayout(); vNet.setSpacing(2); vNet.addLayout(netLine); vNet.addLayout(dbLine)
        h.addLayout(vNet)

        # 4) CPU/RAM mini bar
        self.cpu=self._miniBar(); self.ram=self._miniBar()
        vRes=QVBoxLayout()
        vRes.addWidget(self._wrapBar("CPU",self.cpu))
        vRes.addWidget(self._wrapBar("RAM",self.ram))
        h.addLayout(vRes)

        # 5) E-STOP
        self.estop = PrimaryToolButton(FI.CLOSE, self); self.estop.setText("E-STOP")
        self.estop.setFixedHeight(32); self.estop.setStyleSheet("background:#d9534f;")
        self.estop.clicked.connect(lambda: InfoBar.error("Emergency stop!"))
        h.addWidget(self.estop)

        # spacer
        h.addStretch(1)

        # 6) Avatar + 메뉴
        self.avatar = AvatarWidget("Hanjo", 28, self);  h.addWidget(self.avatar)
        menu=RoundMenu(self.avatar)
        menu.addAction(QAction("Light Theme", self, triggered=lambda: setTheme(Theme.LIGHT)))
        menu.addAction(QAction("Dark Theme",  self, triggered=lambda: setTheme(Theme.DARK )))
        menu.addSeparator()
        menu.addAction(QAction("Logout", self, triggered=lambda: InfoBar.success("Logged out!")))
        self.avatar.setPopupMenu(menu)

        # 7) Search
        self.search = SearchLineEdit(self); self.search.setFixedWidth(260)
        self.search.setPlaceholderText("Search / Ctrl+K")
        h.addWidget(self.search)

        # 타이머
        self._t0=time.time()
        QTimer.singleShot(0, self._updateAll)  # 최초 업데이트
        self.timeT = QTimer(self, timeout=self._updateTime); self.timeT.start(1000)
        self.resT  = QTimer(self, timeout=self._updateRes ); self.resT .start(2000)
        self.netT  = QTimer(self, timeout=self._updateNet ); self.netT .start(5000)

    # ───── 헬퍼들
    def _miniBar(self):
        bar=QProgressBar(self); bar.setFixedSize(80,10)
        bar.setTextVisible(False); return bar
    def _wrapBar(self,txt,bar):
        w=QWidget(self); l=QHBoxLayout(w); l.setContentsMargins(0,0,0,0)
        lbl=QLabel(txt); lbl.setFixedWidth(24); l.addWidget(lbl); l.addWidget(bar,1); return w
    def _load_qss(self):
        qss=os.path.join(os.path.dirname(__file__),"header.qss")
        if os.path.exists(qss):
            with open(qss,encoding="utf-8") as f: self.setStyleSheet(self.styleSheet()+f.read())

    # ───── 알람 테스트
    def _showAlarms(self):
        self.alarm.setBadge(random.randint(0,3))
        InfoBar.info("Alarm list shown (demo)", self)

    # ───── 주기적 업데이트
    def _updateTime(self):
        now=datetime.now()
        self.kst.setText(now.strftime("%H:%M:%S KST"))
        self.utc.setText(datetime.utcnow().strftime("%H:%M:%S UTC"))
        sec=int(time.time()-self._t0); self.timer.setText(f"⏱ {sec//60:02d}:{sec%60:02d}")

    def _updateRes(self):
        self.cpu.setValue(psutil.cpu_percent())
        self.ram.setValue(psutil.virtual_memory().percent)

    def _updateNet(self):
        latency=random.randint(20,120)
        self.netLbl.setText(f"Ping {latency} ms")
        self.netLed.setColor("green" if latency<60 else "orange")
        self.dbLbl.setText("DB OK"); self.dbLed.setColor("green")

    def _updateAll(self):
        self._updateTime(); self._updateRes(); self._updateNet()