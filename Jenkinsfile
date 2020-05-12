pipeline {
  agent {
    node {
      label 'project:any'
    }
  }
  stages {
    stage('Set Build Description') {
      steps {
        script {
          currentBuild.description = "Deploy to ${env.DEPLOY_STAGE}"
        }
      }
    }
    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }
    stage('Git Clone') {
      steps {
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/master']],
            doGenerateSubmoduleConfigurations: false,
            extensions: [],
            submoduleCfg: [],
            userRemoteConfigs: [[credentialsId: 'CIDA-Jenkins-GitHub',
            url: 'https://github.com/gpetrochenkov-usgs/nwc2wmadataprep.git']]])
      }
    }

    stage('Run geoserver config') {
      steps {
        script {
          def wqpSecretsString = sh(script: '/usr/local/bin/aws ssm get-parameter --name "/aws/reference/secretsmanager/WQP-EXTERNAL-$DEPLOY_STAGE" --query "Parameter.Value" --with-decryption --output text --region "us-west-2"', returnStdout: true).trim()

          sh '''
           /usr/local/bin/aws secretsmanager get-secret-value --secret-id IOW-GEOSERVER  --region "us-west-2"
          '''
          def iowGeoSecretsString = sh(script: '/usr/local/bin/aws ssm get-parameter --name "/aws/reference/secretsmanager/IOW-GEOSERVER" --query "Parameter.Value" --with-decryption --output text --region "us-west-2"', returnStdout: true).trim()
          def secretsJson =  readJSON text: secretsString
          env.NWIS_DATABASE_ADDRESS = wqpSecretsString.DATABASE_ADDRESS
          env.NWIS_DATABASE_NAME = wqpSecretsString.DATABASE_NAME
          env.WMADATA_SCHEMA_NAME = wqpSecretsString.WMADATA_SCHEMA_NAME
          env.WMADATA_DB_READ_ONLY_USERNAME = wqpSecretsString.WMADATA_DB_READ_ONLY_USERNAME
          env.WMADATA_DB_READ_ONLY_PASSWORD = wqpSecretsString.WMADATA_DB_READ_ONLY_PASSWORD
          env.POSTGRES_PASSWORD = wqpSecretsString.POSTGRES_PASSWORD
          env.GEOSERVER_PASSWORD = iowGeoSecretsString.admin
          env.GEOSERVER_WORKSPACE = wmadata
          env.GEOSERVER_STORE = wmadata

          sh '''
            if [ $DEPLOY_STAGE == "TEST" ]; then
               echo "TEST"
               url="https://labs-beta.waterdata.usgs.gov/geoserver"
            else
               echo "PROD"
               url="https://labs.waterdata.usgs.gov/geoserver"
            fi

            touch wmadata_store.xml

            echo '<dataStore>
              <name>'$GEOSERVER_STORE'</name>
              <connectionParameters>
                <host>'$NWIS_DATABASE_ADDRESS'</host>
                <port>5432</port>
                <database>'$NWIS_DATABASE_NAME'</database>
                <schema>'WMADATA_SCHEMA_NAME'</schema>
                <user>'$WMADATA_DB_READ_ONLY_USERNAME'</user>
                <passwd>'$WMADATA_DB_READ_ONLY_PASSWORD'</passwd>
                <dbtype>postgis</dbtype>
              </connectionParameters>
            </dataStore>' > wmadata_store.xml

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<workspace><name>$GEOSERVER_WORKSPACE</name></workspace>" \
              $url/rest/workspaces

            curl -v -u admin:geoserver -XPOST -T wmadata_store.xml -H "Content-type: text/xml" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>gagesii</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:900913</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>gagesii</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:900913</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>huc08</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:900913</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>huc12</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:900913</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>huc12all</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:900913</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>huc12agg</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:900913</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>catchmentsp</name><nativeCRS>EPSG:4269</nativeCRS><srs>EEPSG:4269</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>nhdarea</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:4269</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>nhdflowline_network</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:4269</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes

            curl -v -u admin:geoserver -XPOST -H "Content-type: text/xml" \
              -d "<featureType><name>nhdwaterbody</name><nativeCRS>EPSG:4269</nativeCRS><srs>EPSG:4269</srs></featureType>" \
              $url/rest/workspaces/$GEOSERVER_WORKSPACE/datastores/$GEOSERVER_STORE/featuretypes
            '''
        }
      }
    }
  }
}
