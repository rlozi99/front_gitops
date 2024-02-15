# 1. Nginx를 기본 이미지로 선택
FROM nginx:latest

# 2. 작업 디렉토리 설정
WORKDIR /usr/share/nginx/html/

# 3. 현재 디렉토리의 모든 파일을 컨테이너의 작업 디렉토리로 복사
COPY . .

# 4. Nginx 설정 파일을 컨테이너에 복사
COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf

# 5. 80번 포트 개방
EXPOSE 80

ENV LANG=C.UTF-8

# 6. Nginx 실행
CMD ["nginx", "-g", "daemon off;"]
