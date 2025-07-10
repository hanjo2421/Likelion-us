# coding: utf-8
"""
Custom Top HeaderBar  (QFluentWidgets)
──────────────────────────────────────
• Alarm Bell 🔔 • KST / UTC 시계 ⏰ • Ping / DB 상태 🌐🗄
• CPU · RAM 미니 게이지 • 긴급정지 E-STOP • Avatar(메뉴) • Search
"""

import os, sys, psutil, asyncio, time
from datetime import datetime

from PySide6.QtCore    import Qt, QTimer, QSize
from PySide6.QtGui     import QColor, QPainter, QPen, QAction
from PySide6.QtWidgets import (
    QWidget, QLabel, QHBoxLayout, QVBoxLayout, QProgressBar, QStyle
)
from qfluentwidgets import (
    ToolButton, TransparentToolButton, PrimaryToolButton,
    InfoBar, RoundMenu, FluentIcon as FI, AvatarWidget,
    SearchLineEdit, setTheme, Theme
)


# ────────────────────────────────────────────────────────────
# 🔹 작은 LED 점 표시 (빨·초·주·회)
# ────────────────────────────────────────────────────────────
class Dot(QWidget):
    COLORS = {"red": "#D9534F", "green": "#5CB85C",
              "orange": "#F0AD4E", "grey": "#9E9E9E"}

    def __init__(self, color="grey", parent=None):
        super().__init__(parent)
        self._c = color
        self.setFixedSize(10, 10)

    def setColor(self, color): self._c = color; self.update()

    def paintEvent(self, e):
        qp = QPainter(self); qp.setRenderHint(QPainter.Antialiasing)
        qp.setPen(Qt.NoPen); qp.setBrush(QColor(self.COLORS[self._c]))
        r = self.width() / 2 - 1
        qp.drawEllipse(self.rect().center(), r, r)


# ────────────────────────────────────────────────────────────
# 🔹 HeaderBar
# ────────────────────────────────────────────────────────────
class HeaderBar(QWidget):
    """1:8 비율의 얇은(48 px) 상단 헤더"""

    def __init__(self, parent=None):
        super().__init__(parent)
        self.setFixedHeight(48)
        self.setObjectName("HeaderBar")
        self._load_qss()

        h = QHBoxLayout(self)
        h.setContentsMargins(12, 4, 12, 4)
        h.setSpacing(16)

        # ① Alarm Bell
        self.alarm = TransparentToolButton(FI.WARNING, self)
        self.alarm.setBadgeNumber(0)
        self.alarm.setToolTip("Critical / Warning Alarms")
        self.alarm.clicked.connect(self._showAlarmMenu)
        h.addWidget(self.alarm)

        # ② 실시간 시계 (KST / UTC) + 타이머
        self.localLbl = QLabel("--:--:-- KST", self)
        self.utcLbl   = QLabel("--:--:-- UTC", self)
        self.timerLbl = QLabel("⏱  --:--", self)

        vClock = QVBoxLayout()
        vClock.setSpacing(0)
        vClock.addWidget(self.localLbl)
        vClock.addWidget(self.utcLbl)
        h.addLayout(vClock)
        h.addWidget(self.timerLbl)

        # ③ 네트워크·DB 상태
        self.netDot = Dot("grey")
        self.netLbl = QLabel("Ping -- ms", self)
        self.dbDot  = Dot("grey")
        self.dbLbl  = QLabel("DB N/A", self)

        vNet = QVBoxLayout(); vNet.setSpacing(2)
        netLine = QHBoxLayout(); netLine.addWidget(self.netDot); netLine.addWidget(self.netLbl)
        dbLine  = QHBoxLayout(); dbLine.addWidget(self.dbDot);  dbLine.addWidget(self.dbLbl)
        vNet.addLayout(netLine); vNet.addLayout(dbLine)
        h.addLayout(vNet)

        # ④ 리소스 미터
        self.cpuBar = self._miniBar()
        self.ramBar = self._miniBar()
        resBox = QVBoxLayout(); resBox.setSpacing(2)
        resBox.addWidget(self._wrapBar("CPU", self.cpuBar))
        resBox.addWidget(self._wrapBar("RAM", self.ramBar))
        h.addLayout(resBox)

        # ⑤ E-STOP
        self.eStop = PrimaryToolButton(FI.CLOSE, self)
        self.eStop.setFixedSize(80, 32)
        self.eStop.setText("E-STOP")
        self.eStop.clicked.connect(self._confirmStop)
        self.eStop.setStyleSheet("background:#D9534F;")
        h.addWidget(self.eStop)

        # Spacer (유연 비율)
        h.addStretch(1)

        # ⑥ Avatar (Dropdown)
        self.avatar = AvatarWidget("Hanjo", 28, self)
        self.avatar.setToolTip("Engineer")
        self._buildProfileMenu()
        h.addWidget(self.avatar)

        # ⑦ Search
        self.search = SearchLineEdit(self); self.search.setFixedWidth(260)
        self.search.setPlaceholderText("Search / Ctrl+K")
        h.addWidget(self.search)

        # 주기적 업데이트
        self._startTimers()

    # ───────────── 내부 헬퍼 ─────────────
    def _miniBar(self):
        bar = QProgressBar(self)
        bar.setFixedSize(80, 10)
        bar.setTextVisible(False)
        bar.setRange(0, 100)
        return bar

    def _wrapBar(self, label, bar):
        w = QWidget(self); l = QHBoxLayout(w); l.setContentsMargins(0, 0, 0, 0)
        lbl = QLabel(label, self); lbl.setFixedWidth(28)
        l.addWidget(lbl); l.addWidget(bar, 1)
        return w

    def _load_qss(self):
        qss_path = os.path.join(os.path.dirname(__file__), "header.qss")
        if os.path.exists(qss_path):
            with open(qss_path, "r", encoding="utf-8") as f:
                self.setStyleSheet(self.styleSheet() + f.read())

    # ───────────── 메뉴 & 이벤트 ─────────────
    def _buildProfileMenu(self):
        menu = RoundMenu(self)
        act1 = menu.addAction(QAction("Light Theme", self))
        act2 = menu.addAction(QAction("Dark Theme", self))
        menu.addSeparator()
        logout = menu.addAction(QAction("Logout", self))
        act1.triggered.connect(lambda: setTheme(Theme.LIGHT))
        act2.triggered.connect(lambda: setTheme(Theme.DARK ))
        logout.triggered.connect(lambda: InfoBar.success("Logged out!"))
        self.avatar.setPopupMenu(menu)

    def _showAlarmMenu(self):
        modal = RoundMenu(self)
        modal.addAction(QAction("No critical alarms", self))
        modal.exec(self.alarm.mapToGlobal(self.alarm.rect().bottomLeft()))

    def _confirmStop(self):
        InfoBar.error("Emergency Stop Triggered!", duration=3000)

    # ───────────── 주기적 업데이트 ─────────────
    def _startTimers(self):
        timer = QTimer(self); timer.timeout.connect(self._updateTime); timer.start(1000)
        res   = QTimer(self); res.timeout.connect(self._updateResource); res.start(2000)
        net   = QTimer(self); net.timeout.connect(self._updateNet); net.start(5000)

    def _updateTime(self):
        now = datetime.now()
        self.localLbl.setText(now.strftime("%H:%M:%S KST"))
        self.utcLbl.setText(datetime.utcnow().strftime("%H:%M:%S UTC"))
        # 예시:  timerLbl 은 "시스템 가동 경과" → 데모용
        if not hasattr(self, "_t0"):
            self._t0 = time.time()
        sec = int(time.time() - self._t0)
        self.timerLbl.setText(f"⏱ {sec//60:02d}:{sec%60:02d}")

    def _updateResource(self):
        cpu = psutil.cpu_percent()
        ram = psutil.virtual_memory().percent
        self.cpuBar.setValue(cpu)
        self.ramBar.setValue(ram)

    def _updateNet(self):
        # demo: ping 값을 랜덤하게
        import random
        latency = random.randint(18, 80)
        self.netLbl.setText(f"Ping {latency} ms")
        self.netDot.setColor("green" if latency < 60 else "orange")
        # DB 상태도 데모로 OK
        self.dbLbl.setText("DB OK")
        self.dbDot.setColor("green")