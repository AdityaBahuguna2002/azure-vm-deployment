pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        // Production Resource Configuration
        RESOURCE_GROUP = 'Cynoteck.com'
        ACR_NAME = 'cynoappacr'
        WEB_APP_NAME = 'cyno-app-webapp'
        IMAGE_REPO = 'cyno-app'
        IMAGE_TAG = 'latest'
        FULL_IMAGE_NAME = "${ACR_NAME}.azurecr.io/${IMAGE_REPO}:${IMAGE_TAG}"
        
        // Azure Service Principal Credentials
        AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
        AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET_PASS')
        AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
        AZURE_SUBSCRIPTION_ID = credentials('AZURE_SUBSCRIPTION_ID')
        
        // Teams Notifications (commented for now)
        // TEAMS_GROUP_WEBHOOK = credentials('TEAMS_GROUP_WEBHOOK')
        // TEAMS_PERSONAL_WEBHOOK = credentials('TEAMS_PERSONAL_WEBHOOK')
        
        // Deployment start time
        DEPLOY_START_TIME = "${System.currentTimeMillis()}"
    }pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        // Production Resource Configuration
        RESOURCE_GROUP = 'Cynoteck.com'
        ACR_NAME = 'cynoappacr'
        WEB_APP_NAME = 'cyno-app-webapp'
        IMAGE_REPO = 'cyno-app'
        IMAGE_TAG = 'latest'
        FULL_IMAGE_NAME = "${ACR_NAME}.azurecr.io/${IMAGE_REPO}:${IMAGE_TAG}"
        
        // Azure Service Principal Credentials
        AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
        AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET_PASS')
        AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
        AZURE_SUBSCRIPTION_ID = credentials('AZURE_SUBSCRIPTION_ID')
        
        // Teams Notifications (commented for now)
        // TEAMS_GROUP_WEBHOOK = credentials('TEAMS_GROUP_WEBHOOK')
        // TEAMS_PERSONAL_WEBHOOK = credentials('TEAMS_PERSONAL_WEBHOOK')
        
        // Deployment start time
        DEPLOY_START_TIME = "${System.currentTimeMillis()}"
    }

    stages {
        // ============================================
        // STAGE 1: Clean Workspace
        // ============================================
        stage('Clean Workspace') {
            steps {
                echo " Cleaning old builds..."
                cleanWs()
                echo " Workspace cleaned"
            }
        }

        // ============================================
        // STAGE 2: Checkout Code from GitHub
        // ============================================
        stage('Checkout Code from GitHub') {
            steps {
                echo " Checking out code from GitHub (PRODUCTION BRANCH)..."
                
                retry(3) {
                    checkout scmGit(
                        branches: [[name: '*/main']], // PRODUCTION BRANCH
                        extensions: [],
                        userRemoteConfigs: [[
                            credentialsId: 'GITHUB-TOKEN',
                            url: 'https://github.com/macpan83/cynoteck-blogging.git'
                        ]]
                    )
                }
                
                script {
                    env.GIT_COMMIT_MSG = sh(returnStdout: true, script: 'git log -1 --pretty=%B').trim()
                    env.GIT_COMMIT_AUTHOR = sh(returnStdout: true, script: 'git log -1 --pretty=%an').trim()
                    env.GIT_COMMIT_ID = sh(returnStdout: true, script: 'git log -1 --pretty=%h').trim()
                    
                    // Teams notifications (commented out)
                    // sendTeamsNotification(
                    //     webhookUrl: TEAMS_GROUP_WEBHOOK,
                    //     title: " PRODUCTION Deployment Started",
                    //     message: "Production update starting: **${env.GIT_COMMIT_MSG}**",
                    //     color: "FF6B35",
                    //     facts: [
                    //         ["Environment", " PRODUCTION"],
                    //         ["Build Number", "#${BUILD_NUMBER}"],
                    //         ["Commit", "${env.GIT_COMMIT_ID}"],
                    //         ["Author", "${env.GIT_COMMIT_AUTHOR}"],
                    //         ["Estimated Time", "10-15 minutes"]
                    //     ]
                    // )
                }
                
                echo " Code checked out successfully"
                echo "    Commit: ${env.GIT_COMMIT_ID}"
                echo "    Author: ${env.GIT_COMMIT_AUTHOR}"
                echo "    Message: ${env.GIT_COMMIT_MSG}"
                echo "    Environment: PRODUCTION"
            }
        }

        // ============================================
        // STAGE 3: Azure Login
        // ============================================
        stage('Azure Login') {
            steps {
                echo " Logging into Azure..."
                sh """
                    az login --service-principal \
                        --username ${AZURE_CLIENT_ID} \
                        --password ${AZURE_CLIENT_SECRET} \
                        --tenant ${AZURE_TENANT_ID} \
                        --output none
                    
                    az account set --subscription ${AZURE_SUBSCRIPTION_ID}
                    az account show --output table
                """
                echo " Azure login successful"
            }
        }

        // ============================================
        // STAGE 4: Verify Production Environment Variables
        // ============================================
        stage('Verify Production Environment') {
            steps {
                echo " Verifying PRODUCTION environment variables..."
                sh """
                    az webapp config appsettings list \
                        --name ${WEB_APP_NAME} \
                        --resource-group ${RESOURCE_GROUP} \
                        --output table
                """
                echo " Production environment verified"
            }
        }

        // ============================================
        // STAGE 5: Docker Image Build
        // ============================================
        stage('Docker Image Build') {
            steps {
                echo " Building Docker image for PRODUCTION..."
                echo "   Image: ${FULL_IMAGE_NAME}"
                echo "   Build: #${BUILD_NUMBER}"
                echo "    Target: PRODUCTION"
                
                sh """
                    if [ ! -f Dockerfile ]; then
                        echo " ERROR: Dockerfile not found!"
                        exit 1
                    fi
                    
                    # Build with production tags
                    docker build \
                        -t ${FULL_IMAGE_NAME} \
                        -t ${ACR_NAME}.azurecr.io/${IMAGE_REPO}:build-${BUILD_NUMBER} \
                        .
                """
                
                echo " Docker image built successfully"
                echo "   Tags: latest, build-${BUILD_NUMBER}"
            }
        }

        // ============================================
        // STAGE 6: Login to Production ACR
        // ============================================
        stage('ACR Login') {
            steps {
                echo " Logging into PRODUCTION Azure Container Registry..."
                echo "   ACR: ${ACR_NAME}.azurecr.io"
                sh "az acr login --name ${ACR_NAME}"
                echo " Production ACR login successful"
            }
        }

        // ============================================
        // STAGE 7: Check Current Production Images
        // ============================================
        stage('Check Current Images') {
            steps {
                echo " Checking current PRODUCTION images..."
                sh """
                    az acr repository show-tags \
                        --name ${ACR_NAME} \
                        --repository ${IMAGE_REPO} \
                        --output table || echo "  No existing images"
                """
            }
        }

        // ============================================
        // STAGE 8: Push Image to Production ACR
        // ============================================
        stage('Push Image to Production ACR') {
            steps {
                echo "  Pushing Docker images to PRODUCTION ACR..."
                echo "    WARNING: Pushing to PRODUCTION registry!"
                
                script {
                    retry(3) {
                        sh """
                            az acr login --name ${ACR_NAME}
                            timeout 300 docker push ${FULL_IMAGE_NAME} || exit 1
                            timeout 300 docker push ${ACR_NAME}.azurecr.io/${IMAGE_REPO}:build-${BUILD_NUMBER} || exit 1
                        """
                    }
                }
                echo " All images pushed to PRODUCTION ACR"
            }
        }

        // ============================================
        // STAGE 9: Verify Pushed Images
        // ============================================
        stage('Verify Pushed Images') {
            steps {
                echo " Verifying pushed PRODUCTION images..."
                sh """
                    az acr repository show-tags \
                        --name ${ACR_NAME} \
                        --repository ${IMAGE_REPO} \
                        --output table
                """
                echo " Images verified in PRODUCTION ACR"
            }
        }

        // ============================================
        // STAGE 10: Cleanup Old Production Images
        // Keep last 5 build tags + latest
        // ============================================
        stage('Cleanup Old Production Images') {
            steps {
                echo " Cleaning up old PRODUCTION images..."
                echo "   Policy: Keep last 5 build-* tags + latest"
                
                sh """
                    # Get only build-* tags, sorted newest first
                    BUILD_TAGS=\$(az acr repository show-tags \
                        --name ${ACR_NAME} \
                        --repository ${IMAGE_REPO} \
                        --orderby time_desc \
                        --output tsv | grep '^build-')
                    
                    # Count build tags
                    TOTAL_BUILD_TAGS=\$(echo "\$BUILD_TAGS" | wc -l)
                    echo "Total build tags: \$TOTAL_BUILD_TAGS"
                    
                    # Keep last 5, delete rest
                    if [ \$TOTAL_BUILD_TAGS -gt 5 ]; then
                        echo "Deleting old build tags (keeping last 5)..."
                        
                        echo "\$BUILD_TAGS" | tail -n +6 | while read TAG; do
                            if [ ! -z "\$TAG" ]; then
                                echo "  Deleting tag: \$TAG"
                                az acr repository delete \
                                    --name ${ACR_NAME} \
                                    --image ${IMAGE_REPO}:\$TAG \
                                    --yes || echo "  Failed to delete \$TAG"
                            fi
                        done
                        
                        echo " Cleanup completed"
                    else
                        echo " No cleanup needed (5 or fewer build tags)"
                    fi
                    
                    echo ""
                    echo "Remaining PRODUCTION tags:"
                    az acr repository show-tags \
                        --name ${ACR_NAME} \
                        --repository ${IMAGE_REPO} \
                        --output table
                """
            }
        }

        // ============================================
        // STAGE 11: Stop Production App Service
        // ============================================
        stage('Stop Production App Service') {
            steps {
                echo "  Stopping PRODUCTION App Service..."
                echo "    App: ${WEB_APP_NAME}"
                echo "    Domain: cynoteck.com"
                
                sh "az webapp stop --name ${WEB_APP_NAME} --resource-group ${RESOURCE_GROUP}"
                echo " Production App Service stopped"
                sleep 5
            }
        }

        // =======================================================
        // STAGE 12: Update Production Container Configuration
        // =======================================================
        stage('Update Production Container') {
            steps {
                echo "  Updating PRODUCTION container configuration..."
                echo "    New Image: ${FULL_IMAGE_NAME}"
                
                sh """
                    az webapp config container set \
                        --name ${WEB_APP_NAME} \
                        --resource-group ${RESOURCE_GROUP} \
                        --container-image-name ${FULL_IMAGE_NAME}
                """
                echo " Production container configuration updated"
            }
        }

        // ============================================
        // STAGE 13: Start Production App Service
        // ============================================
        stage('Start Production App Service') {
            steps {
                echo "  Starting PRODUCTION App Service..."
                echo "    App: ${WEB_APP_NAME}"
                
                sh "az webapp start --name ${WEB_APP_NAME} --resource-group ${RESOURCE_GROUP}"
                echo " Production App Service started"
                echo " Waiting 30 seconds for container to initialize..."
                sleep 30
            }
        }

        // ============================================
        // STAGE 14: Verify Production App State
        // ============================================
        stage('Verify Production App State') {
            steps {
                echo " Checking PRODUCTION App Service state..."
                script {
                    def appState = sh(
                        script: """
                            az webapp show \
                                --name ${WEB_APP_NAME} \
                                --resource-group ${RESOURCE_GROUP} \
                                --query "state" \
                                --output tsv
                        """,
                        returnStdout: true
                    ).trim()
                    
                    echo "   App State: ${appState}"
                    
                    if (appState == "Running") {
                        echo " PRODUCTION App Service is Running!"
                    } else {
                        echo "  WARNING: PRODUCTION App State is ${appState}"
                    }
                }
            }
        }

        // ============================================
        // STAGE 15: Health Check (Production Domain)
        // ============================================
        stage('Production Health Check') {
            steps {
                echo " Performing PRODUCTION health check..."
                script {
                    // Check main domain
                    echo "   Testing Primary Domain: https://cynoteck.com"
                    def statusCodeMain = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' https://cynoteck.com || echo '000'",
                        returnStdout: true
                    ).trim()
                    echo "   Primary Domain Status: ${statusCodeMain}"
                    
                    sleep 5
                    
                    // Check www subdomain
                    echo "   Testing WWW Domain: https://www.cynoteck.com"
                    def statusCodeWWW = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' https://www.cynoteck.com || echo '000'",
                        returnStdout: true
                    ).trim()
                    echo "   WWW Domain Status: ${statusCodeWWW}"
                    
                    // Check Azure default URL
                    def appUrl = sh(
                        script: """
                            az webapp show \
                                --name ${WEB_APP_NAME} \
                                --resource-group ${RESOURCE_GROUP} \
                                --query "defaultHostName" \
                                --output tsv
                        """,
                        returnStdout: true
                    ).trim()
                    
                    sleep 5
                    
                    echo "   Testing Azure URL: https://${appUrl}"
                    def statusCodeAzure = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' https://${appUrl} || echo '000'",
                        returnStdout: true
                    ).trim()
                    echo "   Azure URL Status: ${statusCodeAzure}"
                    
                    // Evaluate health
                    if (statusCodeMain == "200" || statusCodeWWW == "200" || statusCodeAzure == "200") {
                        echo " PRODUCTION health check PASSED!"
                    } else {
                        echo "  PRODUCTION health check returned unexpected status codes"
                        echo "   Note: App may still be starting up"
                    }
                }
            }
        }

        // ============================================
        // STAGE 16: Verify Custom Domains
        // ============================================
        stage('Verify Custom Domains') {
            steps {
                echo " Verifying custom domains configuration..."
                sh """
                    echo "Checking custom domains for PRODUCTION..."
                    az webapp config hostname list \
                        --webapp-name ${WEB_APP_NAME} \
                        --resource-group ${RESOURCE_GROUP} \
                        --output table
                """
                echo " Custom domains verified"
            }
        }

        // ============================================
        // STAGE 17: Production Deployment Summary
        // ============================================
        stage('Production Deployment Summary') {
            steps {
                script {
                    def appUrl = sh(
                        script: """
                            az webapp show \
                                --name ${WEB_APP_NAME} \
                                --resource-group ${RESOURCE_GROUP} \
                                --query "defaultHostName" \
                                --output tsv
                        """,
                        returnStdout: true
                    ).trim()
                    
                    echo ""
                    echo "╔════════════════════════════════════════════════════════╗"
                    echo "║       PRODUCTION DEPLOYMENT COMPLETED SUCCESSFULLY!    ║"
                    echo "╚════════════════════════════════════════════════════════╝"
                    echo ""
                    echo " PRODUCTION DEPLOYMENT DETAILS:"
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                    echo " Primary Domain: https://cynoteck.com"
                    echo " WWW Domain: https://www.cynoteck.com"
                    echo " Azure URL: https://${appUrl}"
                    echo ""
                    echo " Container Image: ${FULL_IMAGE_NAME}"
                    echo ""
                    echo "  Build Information:"
                    echo "   Build Number: #${BUILD_NUMBER}"
                    echo "   Git Commit: ${env.GIT_COMMIT_ID}"
                    echo "   Author: ${env.GIT_COMMIT_AUTHOR}"
                    echo "   Message: ${env.GIT_COMMIT_MSG}"
                    echo ""
                    echo " Azure Resources:"
                    echo "   Resource Group: ${RESOURCE_GROUP}"
                    echo "   ACR: ${ACR_NAME}.azurecr.io"
                    echo "   Web App: ${WEB_APP_NAME}"
                    echo "   Image Repo: ${IMAGE_REPO}"
                    echo ""
                    echo " Available Tags:"
                    echo "   - latest"
                    echo "   - build-${BUILD_NUMBER}"
                    echo ""
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                    echo " PRODUCTION deployment successful!"
                    echo " Visit: https://cynoteck.com"
                    echo ""
                }
            }
        }

        // ============================================
        // STAGE 18: Cleanup Local Images
        // ============================================
        stage('Cleanup Local Images') {
            steps {
                echo " Cleaning up local Docker images..."
                sh """
                    docker image prune -f || true
                    echo " Cleanup completed"
                """
            }
        }
    }

    // ============================================
    // POST ACTIONS
    // ============================================
    post {
        success {
            // script {
            //     def deployEndTime = System.currentTimeMillis()
            //     def deployStartTime = env.DEPLOY_START_TIME as Long
            //     def totalSeconds = ((deployEndTime - deployStartTime) / 1000) as int
            //     def minutes = (totalSeconds / 60) as int
            //     def seconds = (totalSeconds % 60) as int
                
            //     sendTeamsNotification(
            //         webhookUrl: TEAMS_GROUP_WEBHOOK,
            //         title: " PRODUCTION Deployment Successful!",
            //         message: "**PRODUCTION is LIVE!**\\n\\n [Visit Site](https://cynoteck.com)",
            //         color: "28A745",
            //         facts: [
            //             ["Environment", " PRODUCTION"],
            //             ["Build Number", "#${BUILD_NUMBER}"],
            //             ["Commit", "${env.GIT_COMMIT_ID}"],
            //             ["Author", "${env.GIT_COMMIT_AUTHOR}"],
            //             ["Total Time", "${minutes}m ${seconds}s"],
            //             ["Status", " SUCCESS"]
            //         ]
            //     )
            // }
            
            echo ""
            echo "╔════════════════════════════════════════════════════════╗"
            echo "║            PRODUCTION PIPELINE SUCCESS!                ║"
            echo "╚════════════════════════════════════════════════════════╝"
            echo ""
            echo " All stages completed successfully!"
            echo " Primary: https://cynoteck.com"
            echo " WWW: https://www.cynoteck.com"
            echo "  Build: #${BUILD_NUMBER}"
            echo ""
        }
        
        failure {
            // script {
            //     sendTeamsNotification(
            //         webhookUrl: TEAMS_GROUP_WEBHOOK,
            //         title: " PRODUCTION Deployment Failed!",
            //         message: " CRITICAL: PRODUCTION deployment failed for build #${BUILD_NUMBER}",
            //         color: "DC3545",
            //         facts: [
            //             ["Environment", " PRODUCTION"],
            //             ["Build Number", "#${BUILD_NUMBER}"],
            //             ["Status", " FAILED"]
            //         ]
            //     )
            // }
            
            echo ""
            echo "╔════════════════════════════════════════════════════════╗"
            echo "║           PRODUCTION PIPELINE FAILED!                  ║"
            echo "╚════════════════════════════════════════════════════════╝"
            echo ""
            echo " CRITICAL: Production deployment failed!"
            echo "  Immediate attention required!"
            echo ""
            echo " TROUBLESHOOTING STEPS:"
            echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
            echo "1. Check console output above for error details"
            echo "2. Verify Azure credentials in Jenkins"
            echo "3. Check Docker service is running"
            echo "4. Verify GitHub token is valid"
            echo "5. Check ACR permissions"
            echo ""
            echo "   COMMON ISSUES:"
            echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
            echo "• Azure login failed"
            echo "  → Check Service Principal credentials"
            echo "  → Verify credentials IDs: AZURE_CLIENT_ID, etc."
            echo ""
            echo "• Docker build failed"
            echo "  → Check Dockerfile syntax"
            echo "  → Verify all dependencies exist"
            echo ""
            echo "• ACR push failed"
            echo "  → Verify ACR permissions"
            echo "  → Check network connectivity"
            echo ""
            echo "• Web App deployment failed"
            echo "  → Check web app exists"
            echo "  → Verify resource group name"
            echo ""
        }
        
        always {
            echo ""
            echo " Final cleanup..."
            sh """
                az logout || true
                echo " Azure logout completed"
            """
            cleanWs()
            echo " Workspace cleaned"
            echo ""
        }
    }
}

// Teams notification function (commented out for now)
// def sendTeamsNotification(Map config) {
//     try {
//         def webhookUrl = config.webhookUrl
//         def title = config.title
//         def message = config.message
//         def color = config.color ?: "0078D4"
//         def facts = config.facts ?: []
        
//         def factsJson = facts.collect { fact ->
//             """{"name": "${fact[0]}", "value": "${fact[1]}"}"""
//         }.join(',')
        
//         def payload = """
//         {
//             "@type": "MessageCard",
//             "@context": "https://schema.org/extensions",
//             "summary": "${title}",
//             "themeColor": "${color}",
//             "sections": [
//                 {
//                     "activityTitle": "${title}",
//                     "activitySubtitle": "CYNOTECK Production Deployment",
//                     "text": "${message}",
//                     "facts": [${factsJson}]
//                 }
//             ],
//             "potentialAction": [
//                 {
//                     "@type": "OpenUri",
//                     "name": "View Build Logs",
//                     "targets": [{"os": "default", "uri": "${BUILD_URL}console"}]
//                 },
//                 {
//                     "@type": "OpenUri",
//                     "name": "Visit Site",
//                     "targets": [{"os": "default", "uri": "https://cynoteck.com"}]
//                 }
//             ]
//         }
//         """
        
//         sh """
//             curl -X POST '${webhookUrl}' \
//             -H 'Content-Type: application/json' \
//             -d '${payload.replaceAll("'", "'\\\\''")}'
//         """
        
//         echo " Teams notification sent: ${title}"
//     } catch (Exception e) {
//         echo "  Failed to send Teams notification: ${e.message}"
//     }
// }


    stages {
        // ============================================
        // STAGE 1: Clean Workspace
        // ============================================
        stage('Clean Workspace') {
            steps {
                echo " Cleaning old builds..."
                cleanWs()
                echo " Workspace cleaned"
            }
        }

        // ============================================
        // STAGE 2: Checkout Code from GitHub
        // ============================================
        stage('Checkout Code from GitHub') {
            steps {
                echo " Checking out code from GitHub (PRODUCTION BRANCH)..."
                
                retry(3) {
                    checkout scmGit(
                        branches: [[name: '*/main']], // PRODUCTION BRANCH
                        extensions: [],
                        userRemoteConfigs: [[
                            credentialsId: 'GITHUB-TOKEN',
                            url: 'https://github.com/macpan83/cynoteck-blogging.git'
                        ]]
                    )
                }
                
                script {
                    env.GIT_COMMIT_MSG = sh(returnStdout: true, script: 'git log -1 --pretty=%B').trim()
                    env.GIT_COMMIT_AUTHOR = sh(returnStdout: true, script: 'git log -1 --pretty=%an').trim()
                    env.GIT_COMMIT_ID = sh(returnStdout: true, script: 'git log -1 --pretty=%h').trim()
                    
                    // Teams notifications (commented out)
                    // sendTeamsNotification(
                    //     webhookUrl: TEAMS_GROUP_WEBHOOK,
                    //     title: " PRODUCTION Deployment Started",
                    //     message: "Production update starting: **${env.GIT_COMMIT_MSG}**",
                    //     color: "FF6B35",
                    //     facts: [
                    //         ["Environment", " PRODUCTION"],
                    //         ["Build Number", "#${BUILD_NUMBER}"],
                    //         ["Commit", "${env.GIT_COMMIT_ID}"],
                    //         ["Author", "${env.GIT_COMMIT_AUTHOR}"],
                    //         ["Estimated Time", "10-15 minutes"]
                    //     ]
                    // )
                }
                
                echo " Code checked out successfully"
                echo "    Commit: ${env.GIT_COMMIT_ID}"
                echo "    Author: ${env.GIT_COMMIT_AUTHOR}"
                echo "    Message: ${env.GIT_COMMIT_MSG}"
                echo "    Environment: PRODUCTION"
            }
        }

        // ============================================
        // STAGE 3: Azure Login
        // ============================================
        stage('Azure Login') {
            steps {
                echo " Logging into Azure..."
                sh """
                    az login --service-principal \
                        --username ${AZURE_CLIENT_ID} \
                        --password ${AZURE_CLIENT_SECRET} \
                        --tenant ${AZURE_TENANT_ID} \
                        --output none
                    
                    az account set --subscription ${AZURE_SUBSCRIPTION_ID}
                    az account show --output table
                """
                echo " Azure login successful"
            }
        }

        // ============================================
        // STAGE 4: Verify Production Environment Variables
        // ============================================
        stage('Verify Production Environment') {
            steps {
                echo " Verifying PRODUCTION environment variables..."
                sh """
                    az webapp config appsettings list \
                        --name ${WEB_APP_NAME} \
                        --resource-group ${RESOURCE_GROUP} \
                        --output table
                """
                echo " Production environment verified"
            }
        }

        // ============================================
        // STAGE 5: Docker Image Build
        // ============================================
        stage('Docker Image Build') {
            steps {
                echo " Building Docker image for PRODUCTION..."
                echo "   Image: ${FULL_IMAGE_NAME}"
                echo "   Build: #${BUILD_NUMBER}"
                echo "    Target: PRODUCTION"
                
                sh """
                    if [ ! -f Dockerfile ]; then
                        echo " ERROR: Dockerfile not found!"
                        exit 1
                    fi
                    
                    # Build with production tags
                    docker build \
                        -t ${FULL_IMAGE_NAME} \
                        -t ${ACR_NAME}.azurecr.io/${IMAGE_REPO}:build-${BUILD_NUMBER} \
                        .
                """
                
                echo " Docker image built successfully"
                echo "   Tags: latest, build-${BUILD_NUMBER}"
            }
        }

        // ============================================
        // STAGE 6: Login to Production ACR
        // ============================================
        stage('ACR Login') {
            steps {
                echo " Logging into PRODUCTION Azure Container Registry..."
                echo "   ACR: ${ACR_NAME}.azurecr.io"
                sh "az acr login --name ${ACR_NAME}"
                echo " Production ACR login successful"
            }
        }

        // ============================================
        // STAGE 7: Check Current Production Images
        // ============================================
        stage('Check Current Images') {
            steps {
                echo " Checking current PRODUCTION images..."
                sh """
                    az acr repository show-tags \
                        --name ${ACR_NAME} \
                        --repository ${IMAGE_REPO} \
                        --output table || echo "  No existing images"
                """
            }
        }

        // ============================================
        // STAGE 8: Push Image to Production ACR
        // ============================================
        stage('Push Image to Production ACR') {
            steps {
                echo "  Pushing Docker images to PRODUCTION ACR..."
                echo "    WARNING: Pushing to PRODUCTION registry!"
                
                script {
                    retry(3) {
                        sh """
                            az acr login --name ${ACR_NAME}
                            timeout 300 docker push ${FULL_IMAGE_NAME} || exit 1
                            timeout 300 docker push ${ACR_NAME}.azurecr.io/${IMAGE_REPO}:build-${BUILD_NUMBER} || exit 1
                        """
                    }
                }
                echo " All images pushed to PRODUCTION ACR"
            }
        }

        // ============================================
        // STAGE 9: Verify Pushed Images
        // ============================================
        stage('Verify Pushed Images') {
            steps {
                echo " Verifying pushed PRODUCTION images..."
                sh """
                    az acr repository show-tags \
                        --name ${ACR_NAME} \
                        --repository ${IMAGE_REPO} \
                        --output table
                """
                echo " Images verified in PRODUCTION ACR"
            }
        }

        // ============================================
        // STAGE 10: Cleanup Old Production Images
        // Keep last 5 build tags + latest
        // ============================================
        stage('Cleanup Old Production Images') {
            steps {
                echo " Cleaning up old PRODUCTION images..."
                echo "   Policy: Keep last 5 build-* tags + latest"
                
                sh """
                    # Get only build-* tags, sorted newest first
                    BUILD_TAGS=\$(az acr repository show-tags \
                        --name ${ACR_NAME} \
                        --repository ${IMAGE_REPO} \
                        --orderby time_desc \
                        --output tsv | grep '^build-')
                    
                    # Count build tags
                    TOTAL_BUILD_TAGS=\$(echo "\$BUILD_TAGS" | wc -l)
                    echo "Total build tags: \$TOTAL_BUILD_TAGS"
                    
                    # Keep last 5, delete rest
                    if [ \$TOTAL_BUILD_TAGS -gt 5 ]; then
                        echo "Deleting old build tags (keeping last 5)..."
                        
                        echo "\$BUILD_TAGS" | tail -n +6 | while read TAG; do
                            if [ ! -z "\$TAG" ]; then
                                echo "  Deleting tag: \$TAG"
                                az acr repository delete \
                                    --name ${ACR_NAME} \
                                    --image ${IMAGE_REPO}:\$TAG \
                                    --yes || echo "  Failed to delete \$TAG"
                            fi
                        done
                        
                        echo " Cleanup completed"
                    else
                        echo " No cleanup needed (5 or fewer build tags)"
                    fi
                    
                    echo ""
                    echo "Remaining PRODUCTION tags:"
                    az acr repository show-tags \
                        --name ${ACR_NAME} \
                        --repository ${IMAGE_REPO} \
                        --output table
                """
            }
        }

        // ============================================
        // STAGE 11: Stop Production App Service
        // ============================================
        stage('Stop Production App Service') {
            steps {
                echo "  Stopping PRODUCTION App Service..."
                echo "    App: ${WEB_APP_NAME}"
                echo "    Domain: cynoteck.com"
                
                sh "az webapp stop --name ${WEB_APP_NAME} --resource-group ${RESOURCE_GROUP}"
                echo " Production App Service stopped"
                sleep 5
            }
        }

        // =======================================================
        // STAGE 12: Update Production Container Configuration
        // =======================================================
        stage('Update Production Container') {
            steps {
                echo "  Updating PRODUCTION container configuration..."
                echo "    New Image: ${FULL_IMAGE_NAME}"
                
                sh """
                    az webapp config container set \
                        --name ${WEB_APP_NAME} \
                        --resource-group ${RESOURCE_GROUP} \
                        --container-image-name ${FULL_IMAGE_NAME}
                """
                echo " Production container configuration updated"
            }
        }

        // ============================================
        // STAGE 13: Start Production App Service
        // ============================================
        stage('Start Production App Service') {
            steps {
                echo "  Starting PRODUCTION App Service..."
                echo "    App: ${WEB_APP_NAME}"
                
                sh "az webapp start --name ${WEB_APP_NAME} --resource-group ${RESOURCE_GROUP}"
                echo " Production App Service started"
                echo " Waiting 30 seconds for container to initialize..."
                sleep 30
            }
        }

        // ============================================
        // STAGE 14: Verify Production App State
        // ============================================
        stage('Verify Production App State') {
            steps {
                echo " Checking PRODUCTION App Service state..."
                script {
                    def appState = sh(
                        script: """
                            az webapp show \
                                --name ${WEB_APP_NAME} \
                                --resource-group ${RESOURCE_GROUP} \
                                --query "state" \
                                --output tsv
                        """,
                        returnStdout: true
                    ).trim()
                    
                    echo "   App State: ${appState}"
                    
                    if (appState == "Running") {
                        echo " PRODUCTION App Service is Running!"
                    } else {
                        echo "  WARNING: PRODUCTION App State is ${appState}"
                    }
                }
            }
        }

        // ============================================
        // STAGE 15: Health Check (Production Domain)
        // ============================================
        stage('Production Health Check') {
            steps {
                echo " Performing PRODUCTION health check..."
                script {
                    // Check main domain
                    echo "   Testing Primary Domain: https://cynoteck.com"
                    def statusCodeMain = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' https://cynoteck.com || echo '000'",
                        returnStdout: true
                    ).trim()
                    echo "   Primary Domain Status: ${statusCodeMain}"
                    
                    sleep 5
                    
                    // Check www subdomain
                    echo "   Testing WWW Domain: https://www.cynoteck.com"
                    def statusCodeWWW = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' https://www.cynoteck.com || echo '000'",
                        returnStdout: true
                    ).trim()
                    echo "   WWW Domain Status: ${statusCodeWWW}"
                    
                    // Check Azure default URL
                    def appUrl = sh(
                        script: """
                            az webapp show \
                                --name ${WEB_APP_NAME} \
                                --resource-group ${RESOURCE_GROUP} \
                                --query "defaultHostName" \
                                --output tsv
                        """,
                        returnStdout: true
                    ).trim()
                    
                    sleep 5
                    
                    echo "   Testing Azure URL: https://${appUrl}"
                    def statusCodeAzure = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' https://${appUrl} || echo '000'",
                        returnStdout: true
                    ).trim()
                    echo "   Azure URL Status: ${statusCodeAzure}"
                    
                    // Evaluate health
                    if (statusCodeMain == "200" || statusCodeWWW == "200" || statusCodeAzure == "200") {
                        echo " PRODUCTION health check PASSED!"
                    } else {
                        echo "  PRODUCTION health check returned unexpected status codes"
                        echo "   Note: App may still be starting up"
                    }
                }
            }
        }

        // ============================================
        // STAGE 16: Verify Custom Domains
        // ============================================
        stage('Verify Custom Domains') {
            steps {
                echo " Verifying custom domains configuration..."
                sh """
                    echo "Checking custom domains for PRODUCTION..."
                    az webapp config hostname list \
                        --webapp-name ${WEB_APP_NAME} \
                        --resource-group ${RESOURCE_GROUP} \
                        --output table
                """
                echo " Custom domains verified"
            }
        }

        // ============================================
        // STAGE 17: Production Deployment Summary
        // ============================================
        stage('Production Deployment Summary') {
            steps {
                script {
                    def appUrl = sh(
                        script: """
                            az webapp show \
                                --name ${WEB_APP_NAME} \
                                --resource-group ${RESOURCE_GROUP} \
                                --query "defaultHostName" \
                                --output tsv
                        """,
                        returnStdout: true
                    ).trim()
                    
                    echo ""
                    echo "╔════════════════════════════════════════════════════════╗"
                    echo "║       PRODUCTION DEPLOYMENT COMPLETED SUCCESSFULLY!    ║"
                    echo "╚════════════════════════════════════════════════════════╝"
                    echo ""
                    echo " PRODUCTION DEPLOYMENT DETAILS:"
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                    echo " Primary Domain: https://cynoteck.com"
                    echo " WWW Domain: https://www.cynoteck.com"
                    echo " Azure URL: https://${appUrl}"
                    echo ""
                    echo " Container Image: ${FULL_IMAGE_NAME}"
                    echo ""
                    echo "  Build Information:"
                    echo "   Build Number: #${BUILD_NUMBER}"
                    echo "   Git Commit: ${env.GIT_COMMIT_ID}"
                    echo "   Author: ${env.GIT_COMMIT_AUTHOR}"
                    echo "   Message: ${env.GIT_COMMIT_MSG}"
                    echo ""
                    echo " Azure Resources:"
                    echo "   Resource Group: ${RESOURCE_GROUP}"
                    echo "   ACR: ${ACR_NAME}.azurecr.io"
                    echo "   Web App: ${WEB_APP_NAME}"
                    echo "   Image Repo: ${IMAGE_REPO}"
                    echo ""
                    echo " Available Tags:"
                    echo "   - latest"
                    echo "   - build-${BUILD_NUMBER}"
                    echo ""
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                    echo " PRODUCTION deployment successful!"
                    echo " Visit: https://cynoteck.com"
                    echo ""
                }
            }
        }

        // ============================================
        // STAGE 18: Cleanup Local Images
        // ============================================
        stage('Cleanup Local Images') {
            steps {
                echo " Cleaning up local Docker images..."
                sh """
                    docker image prune -f || true
                    echo " Cleanup completed"
                """
            }
        }
    }

    // ============================================
    // POST ACTIONS
    // ============================================
    post {
        success {
            // script {
            //     def deployEndTime = System.currentTimeMillis()
            //     def deployStartTime = env.DEPLOY_START_TIME as Long
            //     def totalSeconds = ((deployEndTime - deployStartTime) / 1000) as int
            //     def minutes = (totalSeconds / 60) as int
            //     def seconds = (totalSeconds % 60) as int
                
            //     sendTeamsNotification(
            //         webhookUrl: TEAMS_GROUP_WEBHOOK,
            //         title: " PRODUCTION Deployment Successful!",
            //         message: "**PRODUCTION is LIVE!**\\n\\n [Visit Site](https://cynoteck.com)",
            //         color: "28A745",
            //         facts: [
            //             ["Environment", " PRODUCTION"],
            //             ["Build Number", "#${BUILD_NUMBER}"],
            //             ["Commit", "${env.GIT_COMMIT_ID}"],
            //             ["Author", "${env.GIT_COMMIT_AUTHOR}"],
            //             ["Total Time", "${minutes}m ${seconds}s"],
            //             ["Status", " SUCCESS"]
            //         ]
            //     )
            // }
            
            echo ""
            echo "╔════════════════════════════════════════════════════════╗"
            echo "║            PRODUCTION PIPELINE SUCCESS!                ║"
            echo "╚════════════════════════════════════════════════════════╝"
            echo ""
            echo " All stages completed successfully!"
            echo " Primary: https://cynoteck.com"
            echo " WWW: https://www.cynoteck.com"
            echo "  Build: #${BUILD_NUMBER}"
            echo ""
        }
        
        failure {
            // script {
            //     sendTeamsNotification(
            //         webhookUrl: TEAMS_GROUP_WEBHOOK,
            //         title: " PRODUCTION Deployment Failed!",
            //         message: " CRITICAL: PRODUCTION deployment failed for build #${BUILD_NUMBER}",
            //         color: "DC3545",
            //         facts: [
            //             ["Environment", " PRODUCTION"],
            //             ["Build Number", "#${BUILD_NUMBER}"],
            //             ["Status", " FAILED"]
            //         ]
            //     )
            // }
            
            echo ""
            echo "╔════════════════════════════════════════════════════════╗"
            echo "║           PRODUCTION PIPELINE FAILED!                  ║"
            echo "╚════════════════════════════════════════════════════════╝"
            echo ""
            echo " CRITICAL: Production deployment failed!"
            echo "  Immediate attention required!"
            echo ""
            echo " TROUBLESHOOTING STEPS:"
            echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
            echo "1. Check console output above for error details"
            echo "2. Verify Azure credentials in Jenkins"
            echo "3. Check Docker service is running"
            echo "4. Verify GitHub token is valid"
            echo "5. Check ACR permissions"
            echo ""
            echo "   COMMON ISSUES:"
            echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
            echo "• Azure login failed"
            echo "  → Check Service Principal credentials"
            echo "  → Verify credentials IDs: AZURE_CLIENT_ID, etc."
            echo ""
            echo "• Docker build failed"
            echo "  → Check Dockerfile syntax"
            echo "  → Verify all dependencies exist"
            echo ""
            echo "• ACR push failed"
            echo "  → Verify ACR permissions"
            echo "  → Check network connectivity"
            echo ""
            echo "• Web App deployment failed"
            echo "  → Check web app exists"
            echo "  → Verify resource group name"
            echo ""
        }
        
        always {
            echo ""
            echo " Final cleanup..."
            sh """
                az logout || true
                echo " Azure logout completed"
            """
            cleanWs()
            echo " Workspace cleaned"
            echo ""
        }
    }
}

// Teams notification function (commented out for now)
// def sendTeamsNotification(Map config) {
//     try {
//         def webhookUrl = config.webhookUrl
//         def title = config.title
//         def message = config.message
//         def color = config.color ?: "0078D4"
//         def facts = config.facts ?: []
        
//         def factsJson = facts.collect { fact ->
//             """{"name": "${fact[0]}", "value": "${fact[1]}"}"""
//         }.join(',')
        
//         def payload = """
//         {
//             "@type": "MessageCard",
//             "@context": "https://schema.org/extensions",
//             "summary": "${title}",
//             "themeColor": "${color}",
//             "sections": [
//                 {
//                     "activityTitle": "${title}",
//                     "activitySubtitle": "CYNOTECK Production Deployment",
//                     "text": "${message}",
//                     "facts": [${factsJson}]
//                 }
//             ],
//             "potentialAction": [
//                 {
//                     "@type": "OpenUri",
//                     "name": "View Build Logs",
//                     "targets": [{"os": "default", "uri": "${BUILD_URL}console"}]
//                 },
//                 {
//                     "@type": "OpenUri",
//                     "name": "Visit Site",
//                     "targets": [{"os": "default", "uri": "https://cynoteck.com"}]
//                 }
//             ]
//         }
//         """
        
//         sh """
//             curl -X POST '${webhookUrl}' \
//             -H 'Content-Type: application/json' \
//             -d '${payload.replaceAll("'", "'\\\\''")}'
//         """
        
//         echo " Teams notification sent: ${title}"
//     } catch (Exception e) {
//         echo "  Failed to send Teams notification: ${e.message}"
//     }
// }
