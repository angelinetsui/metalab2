# Set up instructions

#### 0. These instructions assume an Ubuntu 16.04 OS

#### 1. Create a passwordless `metalab` user with sudo privilege

```
adduser metalab
passwd -d metalab
gpasswd -a metalab sudo
```

Allow `metalab` user to run sudo commands without password (so that the automatic build script can run sudo) by opening the sudoers file with the `visudo` command and add the following line to the end of the file:

```
metalab ALL=(ALL) NOPASSWD: ALL
```

Now switch users to the metalab user:

```
su - metalab
```

#### 2. Install R

See [here for instructions](http://deanattali.com/2015/05/09/setup-rstudio-shiny-server-digital-ocean/) (step 6) and also install the `purrr`, `dplyr`, and `shiny` packages:
```
sudo su - -c "R -e \"install.packages(c('purrr', 'dplyr', 'shiny'), repos='http://cran.rstudio.com/')\""
```

For the `nloptr` R package (that is required by one of the Rmd reports), you need to run the following command:

```
sudo apt-get install libnlopt-dev
```

#### 3. Install shiny server

```
sudo apt-get install gdebi-core
wget https://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.4.2.786-amd64.deb
sudo gdebi shiny-server-1.4.2.786-amd64.deb
```

Give metalab user permission to write to the shiny apps folder:

```
sudo chown metalab -R /srv/shiny-server/
```
 
#### 4. Install pandoc

```
sudo wget https://github.com/jgm/pandoc/releases/download/1.19.2.1/pandoc-1.19.2.1-1-amd64.deb
sudo dpkg -i pandoc-1.19.2.1-1-amd64.deb
```

#### 5. Clone repository into `/home/metalab/` directory

```
sudo apt-get -y install git
cd /home/metalab
git clone https://metalabgithub:<token>@github.com/daattali/metalab2.git`
```  

Make sure to use the correct git repository name in the `git clone` command. **IMPORTANT**: Also make sure the `scripts/deploy.sh` script has the correct git repository name if you change it.   

The <token> is the PAT (personal access token) for the `metalabgithub` GitHub user.
You can create a GitHub Personal Access Token at https://github.com/settings/tokens and make sure to select the "repo" scope.

#### 6. Set git info

```
git config --global user.email "metalab.github@gmail.com"
git config --global user.name "Metalab Cron"
```

#### 7. Copy `/scripts/deploy.sh` script into `/home/metalab/` and add permissions to it

```
cd /home/metalab/metalab2/
cp scripts/deploy.sh ../deploy.sh
sudo chmod u+x ../deploy.sh
```

#### 8. Update `serverAddress` variable at `metalab2/pages/assets/helpers.R`with YOUR server address

#### 9. Set up crontab to automatically deploy the site:
 
```
crontab -e
```

and add the following line:
`0 0 * * * /home/metalab/deploy.sh > /home/metalab/daily-backup.log 2>&1`

---

# Next steps and useful info

**VERY IMPORTANT** Make sure you only edit using `master` branch (or by submitting PRs into `master`). Do not manually edit `gh-pages` branch because it will get overwritten.

Any change you make in `master` will be reflected in the site the next time the script runs (within 24 hours).

### Scripts

The `scripts/deploy.sh` script is executed automatically every day and is responsible for:

1. Making `master` branch up to date with remote (GitHub version).
2. Installing and updating R packages (using the `scripts/packageInstaller.R` script).
3. Rendering all HTML files (all pages) and copying shinyapps to the server directory on the machine (using the `scripts/main_builder.R` script).
4. Preparing `gh-pages` branch to push, and pushing it to remote.

You generally do not need to touch the scripts folder.

Every time the script runs, it will generate a log file at `/home/metalab/daily-backup.log`.

### Package management.
All packages that you need to have installed for the Rmarkdown or shiny apps need to be listed in `metadata/packageInfo.csv`. 

File specification:

  - `Package` - package name
  - `Version` - version of needed package or `NA` when newest version should be installed
  - `Repo` - `cran` or `github`
  - `RepoName` - `NA` when `Repo` is cran, `<github_user>/<github_repo_name>` when `Repo` is github

Whenever this file gets modified, the next build of the site (within 24 hours) will use the new packages.

### Metadata folder

The `metadata` folder contains many list-type data that is used throughout the site. To change it, simply edit the corresponding file in the `metadata` folder and wait for the site to render again. Use these metadata files whenever you add or edit a dataset, a domain, a shiny app, a tutorial tab, a documentation tab, a person in the about page.

### Adding new or editing documentation/tutorial/report tabs

Each tab in these 3 pages is an Rmd file under the `documentation`, `tutorials`, and `reports` folders, respectively. Simply add or edit an Rmd there. If you're adding a new tab or want to change the name of the tab, you need to edit the corresponding metadata file in the `metadata` folder.

### Shiny apps

To add a new shiny app or edit a shiny app, look in the `shinyapps` folder. Don't forget to edit the corresponding metadata file.

All current shiny apps use a common R file `shinyapps/common/global.R`. You can access it (source it) using `source('../common/global.R')`. Variables that are currently accessible from this file:  

    - project_directory - direct path to `metalab2` repository  
    - fields  
    - fields_derived  
    - datasets_file  
    - datasets    
    - cached_data  
    - avg_month   
    - all_data  
    - studies   
    - subjects   
    - datasets  

If any new packages are required, update the `metadata/packageInfo.csv` file.

### Editing the other pages

The code for the other pages is in the `pages/` folder.

### Images

For better organization, try to place all images under the `images/` folder in the correct subfolder, and use that path in metadata files when referring to an image.
