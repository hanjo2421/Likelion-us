# LIKELION US Web & Mentoring Platform

이 프로젝트는 LIKELION US 웹페이지 및 멘토링 플랫폼을 위한 React 프로젝트입니다.

## 📌 프로젝트 실행 방법

### **1. 저장소 클론**
```bash
git clone https://github.com/hanjo2421/Likelion-us.git
cd Likelion-us/frontend/web # 웹페이지 실행 시
cd Likelion-us/frontend/mentoring # 멘토링 플랫폼 실행 시
```

### **2. 패키지 설치**
패키지를 설치하려면 아래 명령어를 실행하세요.
```bash
npm install
```

### **3. 개발 서버 실행**
프로젝트를 실행하려면 아래 명령어를 사용하세요.
```bash
npm start
```
이후 브라우저에서 확인할 수 있습니다.
- **웹페이지:** [http://localhost:3000](http://localhost:3000)
- **멘토링 플랫폼:** [http://localhost:3001](http://localhost:3001)

---

## 📌 프로젝트 구조
```
frontend/
├── web/           # LIKELION US 웹페이지
│   ├── public/    # 정적 파일 (favicon, index.html 등)
│   ├── src/       # React 소스 코드
│   ├── package.json  # 프로젝트 정보 및 의존성 관리
│   ├── README.md  # 프로젝트 설명
├── mentoring/     # LIKELION US 멘토링 플랫폼
│   ├── public/    # 정적 파일 (favicon, index.html 등)
│   ├── src/       # React 소스 코드
│   ├── package.json  # 프로젝트 정보 및 의존성 관리
│   ├── README.md  # 프로젝트 설명
```

---

## 📌 주요 스크립트

### **📺 패키지 설치**
```bash
npm install
```

### **🚀 개발 서버 실행**
```bash
npm start
```
- **웹페이지:** [http://localhost:3000](http://localhost:3000)
- **멘토링 플랫폼:** [http://localhost:3001](http://localhost:3001)

### **🛠 빌드 (배포용)**
```bash
npm run build
```
- `build/` 폴더에 최적화된 정적 파일이 생성됩니다.

### **🔍 코드 스타일 확인 (ESLint)**
```bash
npm run lint
```

---

## 📌 Git 관리
### **커밋 & 푸시**
```bash
git add .
git commit -m "Update project"
git push origin main
```

### **원격 저장소 최신 코드 가져오기**
```bash
git pull origin main --rebase
```

---

## 📌 FAQ

### **1. `npm start` 실행 시 오류가 발생해요.**
**해결 방법:**
```bash
rm -rf node_modules package-lock.json
npm cache clean --force
npm install
npm start
```

### **2. 포트가 이미 사용 중이라는 오류가 떠요.**
```bash
Error: listen EADDRINUSE: address already in use 127.0.0.1:3000
```
**해결 방법:**
```bash
kill -9 $(lsof -t -i:3000)
npm start
```

### **3. 패키지 충돌로 `npm install`이 안 돼요.**
**해결 방법:**
```bash
npm install --legacy-peer-deps
npm install --force
```

---

## 📌 팀원 가이드
- PR(Pull Request) 기반으로 코드 리뷰 후 병합
- `main` 브랜치는 **항상 배포 가능한 상태로 유지**
- 새로운 기능 추가 시 **feature 브랜치**에서 작업 후 PR 작성
```bash
git checkout -b feature/new-feature
```

---

## 📌 라이선스
이 프로젝트는 MIT 라이선스를 따릅니다.

