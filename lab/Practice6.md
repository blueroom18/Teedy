说明：使 mvn site 正常工作的 pom.xml 修改                                                                                                              
                                                                                                                                                         
  为了让 mvn site 命令正常工作，我在父 pom.xml 中进行了以下修改：                                                                                        
                                                                                                                                                         
  1. 添加 Maven Site 插件版本和配置                                                                                                                      
                                                                                                                                                         
  <!-- 属性定义 -->                                                                                                                                      
  <maven-site-plugin.version>3.12.1</maven-site-plugin.version>                                                                                          
  <maven-project-info-reports-plugin.version>3.4.2</maven-project-info-reports-plugin.version>                                                           
  <maven-jxr-plugin.version>3.3.0</maven-jxr-plugin.version>                                                                                             
                                                                                                                                                         
  <!-- Build 插件配置 -->                                                                                                                                
  <plugin>                                                                                                                                               
    <groupId>org.apache.maven.plugins</groupId>                                                                                                          
    <artifactId>maven-site-plugin</artifactId>                                                                                                           
    <version>${maven-site-plugin.version}</version>                                                                                                      
    <executions>                                                                                                                                         
      <execution>                                                                                                                                        
        <id>attach-descriptor</id>                                                                                                                       
        <goals>                                                                                                                                          
          <goal>attach-descriptor</goal>                                                                                                                 
        </goals>                                                                                                                                         
      </execution>                                                                                                                                       
    </executions>                                                                                                                                        
  </plugin>                                                                                                                                              
                                                                                                                                                         
  2. 添加 <reporting> 部分配置报告生成                                                                                                                   
                                                                                                                                                         
  <reporting>                                                                                                                                            
    <plugins>                                                                                                                                            
      <plugin>                                                                                                                                           
        <groupId>org.apache.maven.plugins</groupId>                                                                                                      
        <artifactId>maven-project-info-reports-plugin</artifactId>                                                                                       
        <reports>                                                                                                                                        
          <report>index</report>                                                                                                                         
          <report>dependencies</report>                                                                                                                  
          <report>dependency-convergence</report>                                                                                                        
          <report>dependency-management</report>                                                                                                         
          <report>plugins</report>                                                                                                                       
          <report>summary</report>                                                                                                                       
          <report>modules</report>                                                                                                                       
        </reports>                                                                                                                                       
      </plugin>                                                                                                                                          
      <plugin>                                                                                                                                           
        <groupId>org.apache.maven.plugins</groupId>                                                                                                      
        <artifactId>maven-jxr-plugin</artifactId>                                                                                                        
        <reports>                                                                                                                                        
          <report>jxr</report>                                                                                                                           
        </reports>                                                                                                                                       
      </plugin>                                                                                                                                          
    </plugins>                                                                                                                                           
  </reporting>                                                                                                                                           
                                                                                                                                                         
  3. 修复依赖收敛问题                                                                                                                                    
                                                                                                                                                         
  在 <dependencyManagement> 中添加了多个传递依赖的版本管理（如 commons-logging、jakarta.activation-api、checker-qual、kotlin-stdlib 等），使 Dependency  
  Convergence 达到 100%。                                                                                                                                
                                                                                                                                                         
  4. 在子模块中添加依赖排除                                                                                                                              
                                                                                                                                                         
  在 docs-web-common/pom.xml 和 docs-web/pom.xml 中为 subethasmtp-wiser 依赖添加了 commons-logging 的排除，解决版本冲突问题。   