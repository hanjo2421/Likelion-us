# LIKELION US Web

이 프로젝트는 LIKELION US 웹페이지를 위한 React 프로젝트입니다.

## 📌 프로젝트 실행 방법

### **1. 저장소 클론**
```bash
git clone https://github.com/hanjo2421/Likelion-us.git
cd Likelion-us/frontend/web
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
이후 브라우저에서 [http://localhost:3000](http://localhost:3000)로 접속하여 확인할 수 있습니다.

---

## 📌 프로젝트 구조
```
frontend/web/
├── public/        # 정적 파일 (favicon, index.html 등)
├── src/           # React 소스 코드
│   ├── components/ # 컴포넌트 모음
│   ├── pages/      # 개별 페이지
│   ├── App.js      # 메인 컴포넌트
│   ├── index.js    # React 진입점
│   ├── styles/     # CSS 스타일링
│   ├── assets/     # 이미지, 폰트 등
├── .gitignore      # Git에서 무시할 파일 목록
├── package.json    # 프로젝트 정보 및 의존성 관리
├── package-lock.json # 패키지 버전 고정 파일
├── README.md       # 프로젝트 설명
```

---

## 📌 주요 스크립트

### **📺 패키지 설치**
```bash
npm install
```
- `node_modules/`가 없을 경우 패키지를 설치합니다.

### **🚀 개발 서버 실행**
```bash
npm start
```
- 개발 서버를 실행하고 [http://localhost:3000](http://localhost:3000)에서 확인할 수 있습니다.

### **🛠 빌드 (배포용)**
```bash
npm run build
```
- `build/` 폴더에 최적화된 정적 파일이 생성됩니다.

### **🔍 코드 스타일 확인 (ESLint)**
```bash
npm run lint
```

### **🚀 프로젝트 배포**
배포 환경에 따라 다르지만, 기본적으로 `build/` 폴더를 업로드하면 됩니다.
```bash
npm run build
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
1. `node_modules/`와 `package-lock.json`을 삭제한 후 다시 설치합니다.
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
1. 실행 중인 프로세스를 종료합니다.
   ```bash
   kill -9 $(lsof -t -i:3000)
   ```
2. 다시 실행합니다.
   ```bash
   npm start
   ```

### **3. 패키지 충돌로 `npm install`이 안 돼요.**
**해결 방법:**
1. `--legacy-peer-deps` 옵션을 추가하여 강제 설치합니다.
   ```bash
   npm install --legacy-peer-deps
   ```
2. 그래도 안 될 경우 `--force` 옵션을 추가합니다.
   ```bash
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
