pipeline{
agent any
tools{
maven 'Maven'
}
stages{
stage('Checkout'){
steps{
git branch:'master',url:'https://github.com/varshini663/FinMav.git'
}
}
stage('Build'){
steps{
sh 'mvn clean package'
}
}
stage('Test'){
steps{
sh 'mvn test'
}
}
stage('Run'){
steps{
sh 'java -jar target/FinMav-1.0-SNAPSHOT.jar'
}
}
}
post{
success{
echo 'Successful'
}
failure{
echo 'not built'
}
}
}
