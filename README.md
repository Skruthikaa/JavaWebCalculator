<hr>
<h1>CI/CD Pipeline Setup using Jenkins, SonarQube, Nexus &amp; Tomcat on AWS EC2</h1>
<p>This guide walks through creating a <strong>complete end-to-end CI/CD pipeline</strong> using <strong>Jenkins, SonarQube, Nexus, and Tomcat</strong>, each hosted on a <strong>dedicated AWS EC2 instance</strong>.<br>
All tools are Java-based, so ensure <strong>Java 21</strong> is installed on all servers.</p>
<hr>
<h2> 1. Create EC2 Instance for Jenkins</h2>
<h3> Configuration</h3>
<ul>
<li>
<p><strong>AMI:</strong> Ubuntu 22.04 LTS</p>
</li>
<li>
<p><strong>Instance Type:</strong> t2.medium</p>
</li>
<li>
<p><strong>Storage:</strong> 20 GB</p>
</li>
<li>
<p><strong>Ports to Open:</strong> 22 (SSH), 8080 (Jenkins)</p>
</li>
</ul>
<h3 Steps</h3>
<ol>
<li>
<p>Launch the instance and connect via SSH.</p>
</li>
<li>
<p>Configure the key pair, security group, and storage.</p>
</li>
</ol>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/cb2fb41e-a7d2-4e6e-94aa-9a6eb21fda9b" alt="Jenkins EC2 Setup"><br>
<img src="https://github.com/user-attachments/assets/0e77f762-275a-4503-8052-cffe56fa919c" alt="Jenkins EC2 Connected"></p>
<hr>
<h2>2. Create EC2 Instance for SonarQube</h2>
<h3>Configuration</h3>
<ul>
<li>
<p><strong>AMI:</strong> Ubuntu 22.04 LTS</p>
</li>
<li>
<p><strong>Instance Type:</strong> c7i-flex.large</p>
</li>
<li>
<p><strong>Ports:</strong> 22 (SSH), 9000 (SonarQube)</p>
</li>
</ul>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/f88c7064-0b22-4056-9755-5b9b2f63ff50" alt="Sonar EC2 Setup"><br>
<img src="https://github.com/user-attachments/assets/0cd88cfa-6514-4835-aeb8-02a55fdc9abe" alt="Sonar EC2 Connected"></p>
<hr>
<h2>3. Create EC2 Instance for Nexus Repository</h2>
<h3>Configuration</h3>
<ul>
<li>
<p><strong>Ports:</strong> 22 (SSH), 8081 (Nexus)</p>
</li>
</ul>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/009de996-a330-4cdd-acc4-c33f96a6baff" alt="Nexus EC2 Setup"><br>
<img src="https://github.com/user-attachments/assets/33b4ea18-10f3-4887-9589-4ee3823284ec" alt="Nexus EC2 Connected"></p>
<hr>
<h2>4. Create EC2 Instance for Tomcat</h2>
<h3>Configuration</h3>
<ul>
<li>
<p><strong>Ports:</strong> 22 (SSH), 8080 (Tomcat)</p>
</li>
</ul>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/d6248988-da9d-461d-aa4f-e281095b1777" alt="Tomcat EC2 Setup"><br>
<img src="https://github.com/user-attachments/assets/d2da0874-c4b3-47d4-b4a6-309236223e2a" alt="Tomcat EC2 Connected"></p>
<hr>
<h2>5. Install Java (OpenJDK 21) on All Servers</h2>
<pre><code class="language-bash">sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
java -version
</code></pre>
<p>Expected output:</p>
<pre><code>openjdk 21.0.8 2025-07-15
OpenJDK Runtime Environment (build 21.0.8+9-Debian-1)
OpenJDK 64-Bit Server VM (build 21.0.8+9-Debian-1, mixed mode, sharing)
</code></pre>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/7cbfc4d8-83e0-4b09-a3e7-64241ad63812" alt="Java Installation"><br>
<img src="https://github.com/user-attachments/assets/d3e68775-c31d-4250-8aed-46c88f2d288c" alt="Java Verification"></p>
<hr>
<h2>6. Install Jenkins</h2>
<pre><code class="language-bash">sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list &gt; /dev/null
sudo apt update
sudo apt install jenkins -y
</code></pre>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/29aecac3-0758-4192-b04b-42e3a19c26b5" alt="Jenkins Installation"><br>
<img src="https://github.com/user-attachments/assets/ad53d0b1-8cfc-47fa-ab98-f2cc93fee242" alt="Jenkins Running"></p>
<hr>
<h2>7. Setup SonarQube Server</h2>
<pre><code class="language-bash">cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.6.0.92116.zip
sudo unzip sonarqube-10.6.0.92116.zip
sudo mv sonarqube-10.6.0.92116 /opt/sonarqube
sudo useradd -r -s /bin/false sonar
sudo chown -R sonar:sonar /opt/sonarqube
</code></pre>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/a546c32d-f5e9-407b-a72d-bfe9b64c1078" alt="SonarQube Setup"><br>
<img src="https://github.com/user-attachments/assets/23d88e81-d9a5-413a-9d71-17358df2a0c4" alt="SonarQube Token"></p>
<hr>
<h2>8. Setup Nexus Repository</h2>
<pre><code class="language-bash">cd /opt
wget https://download.sonatype.com/nexus/3/nexus-3.85.0-03-linux-x86_64.tar.gz
sudo tar -xzf nexus-3.85.0-03-linux-x86_64.tar.gz
sudo mv nexus-3* nexus
sudo useradd -r -s /bin/false nexus
sudo chown -R nexus:nexus /opt/nexus /opt/sonatype-work
</code></pre>
<p><strong>Screenshot:</strong><br>
<img src="https://github.com/user-attachments/assets/aad22e28-33c1-47c2-801a-eb92dc7a3e6c" alt="Nexus Setup"></p>
<hr>
<h2>9. Setup Tomcat Server</h2>
<pre><code class="language-bash">sudo apt update
sudo apt install -y tomcat9 tomcat9-admin
</code></pre>
<h3>Add User in <code inline="">/etc/tomcat9/tomcat-users.xml</code></h3>
<pre><code class="language-xml">&lt;role rolename="manager-gui"/&gt;
&lt;role rolename="manager-script"/&gt;
&lt;user username="admin" password="admin123" roles="manager-gui,manager-script"/&gt;
</code></pre>
<p><strong>Screenshot:</strong><br>
<img src="https://github.com/user-attachments/assets/ba989e9a-5a30-4b55-be06-9744ef220a61" alt="Tomcat Setup"></p>
<hr>
<h2>10. Configure SSH Key Access from Jenkins to All Servers</h2>
<pre><code class="language-bash">sudo su - jenkins
ssh-keygen
cat ~/.ssh/id_rsa.pub
</code></pre>
<p>Copy the public key to each server’s <code inline="">~/.ssh/authorized_keys</code>.</p>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/a261045d-ee54-4f19-ba75-4f32643fd310" alt="SSH Key Generation"><br>
<img src="https://github.com/user-attachments/assets/4e11f40d-a2cf-4e3b-8434-1e8d7b29554e" alt="SSH Verification"></p>
<hr>
<h2>11. Configure Jenkins</h2>
<h3>a. Install Required Plugins</h3>
<ul>
<li>
<p>Git Plugin</p>
</li>
<li>
<p>GitHub Integration</p>
</li>
<li>
<p>Pipeline</p>
</li>
<li>
<p>Maven Integration</p>
</li>
<li>
<p>SonarQube Scanner for Jenkins</p>
</li>
<li>
<p>SSH Slaves</p>
</li>
<li>
<p>Credentials Binding</p>
</li>
</ul>
<p><strong>Screenshot:</strong><br>
<img src="https://github.com/user-attachments/assets/cb979968-5bf6-4ac3-b047-2fad8ef7efdf" alt="Jenkins Plugins"></p>
<hr>
<h3>b. Global Tool Configuration</h3>
<ul>
<li>
<p><strong>JDK Name:</strong> <code inline="">JDK17</code></p>
</li>
<li>
<p><strong>Maven Name:</strong> <code inline="">Maven</code></p>
</li>
</ul>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/7fedb3e4-13d8-48ca-b78b-68cbcfd4ea12" alt="JDK Configuration"><br>
<img src="https://github.com/user-attachments/assets/e4cd8980-7c33-4388-b963-1a423ef6c194" alt="Maven Configuration"></p>
<hr>
<h3>c. Add Credentials</h3>
<ul>
<li>
<p><strong>SSH Username with Private Key</strong> → for Ubuntu agents</p>
</li>
<li>
<p><strong>Nexus Credentials</strong></p>
</li>
<li>
<p><strong>Tomcat Manager Credentials</strong></p>
</li>
</ul>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/9f53c2c7-5ea5-427e-8cb8-c1eb8a39197e" alt="Jenkins Credentials"></p>
<hr>
<h2>12. Configure SonarQube in Jenkins</h2>
<p><strong>Path:</strong> <code inline="">Manage Jenkins → Configure System → SonarQube servers</code></p>

Field | Value
-- | --
Name | SonarQube Server
URL | http://SONAR_IP:9000
Authentication Token |  


<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/b8808bcb-b049-4b42-b715-f86d418ff265" alt="Sonar Node"><br>
<img src="https://github.com/user-attachments/assets/28e44cda-58dd-4dd5-b19b-4a87f6d8d193" alt="Nexus Node"><br>
<img src="https://github.com/user-attachments/assets/9798bdd8-6f2f-449d-8753-a2bbb015fb27" alt="Tomcat Node"></p>

<img width="1919" height="678" alt="Image" src="https://github.com/user-attachments/assets/ac9c11a6-84b8-4cea-ad84-1f10f1013973" />

<hr>
<h2>14. Create Jenkins Pipeline Job</h2>
<ol>
<li>
<p><strong>Jenkins → New Item → Pipeline</strong></p>
</li>
<li>
<p><strong>Definition:</strong> Pipeline script from SCM</p>
</li>
<li>
<p><strong>Repository URL:</strong> <code inline="">https://github.com/you/your-app.git</code></p>
</li>
<li>
<p><strong>Branch:</strong> <code inline="">*/main</code></p>
</li>
</ol>
<p><strong>Screenshots:</strong><br>

<img width="1866" height="766" alt="Image" src="https://github.com/user-attachments/assets/0e55c019-9937-4c17-abb6-333a0419a270" />

<img width="1913" height="751" alt="Image" src="https://github.com/user-attachments/assets/2ec4c02a-d78d-429c-bd1c-be8617234c2d" />

<img width="1829" height="901" alt="Image" src="https://github.com/user-attachments/assets/2c8fda0b-f081-41a2-ae87-1a72505bfe90" />

<img src="https://github.com/user-attachments/assets/80e67337-f0f1-4895-bc53-73c599f612a0" alt="Pipeline Config"></p>
<hr>
<h2>15. Verify Deployment on Tomcat</h2>
<p>After successful build, refresh Tomcat at:<br>
<code inline="">http://&lt;Tomcat-IP&gt;:8080/manager/html</code></p>
<p><strong>Screenshots:</strong><br>
<img src="https://github.com/user-attachments/assets/06e455b9-6acc-41a1-a181-fa1f0fecd65c" alt="Deployed App"><br>
<img src="https://github.com/user-attachments/assets/b60b6da1-b40c-4f91-b355-56dbcacd2e2a" alt="App Output"></p>
<hr>
<h2>Final CI/CD Flow Summary</h2>
<ol>
<li>
<p><strong>Jenkins</strong> → Pulls Code from GitHub</p>
</li>
<li>
<p><strong>SonarQube</strong> → Performs Code Quality Analysis</p>
</li>
<li>
<p><strong>Maven</strong> → Builds &amp; Packages the Application</p>
</li>
<li>
<p><strong>Nexus</strong> → Stores the Build Artifact</p>
</li>
<li>
<p><strong>Tomcat</strong> → Deploys the Final Application</p>
</li>
</body>
</html>
