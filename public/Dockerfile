# Node.js גרסה 18
FROM node:18-alpine

# תיקיית העבודה בתוך הקונטיינר
WORKDIR /usr/src/app

# מעתיק קבצי ההגדרות של החבילות
COPY package*.json ./

# מתקין את התלויות של הפרויקט
RUN npm ci --only=production

# מעתיק את שאר קבצי הפרויקט
COPY . .

# חשוף את פורט 3000
EXPOSE 3000

# הגדרת משתנה סביבה לפורט
ENV PORT=3000

# הפקודה להרצת האפליקציה
CMD ["npm", "start"]