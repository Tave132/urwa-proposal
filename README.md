# Make a fresh folder for Pages
mkdir urwa-proposal-pages
cd urwa-proposal-pages

# Copy in your static build
cp -r ../proposal-site/public/* .
index.html  story.html  gallery.html  proposal.html  countdown.html  
css/        js/         assets/  
git init
git branch -M main
git remote add origin git@github.com:<your-username>/urwa-proposal.git
git add .
git commit -m "Initial static site for Urwa proposal"
git push -u origin main
