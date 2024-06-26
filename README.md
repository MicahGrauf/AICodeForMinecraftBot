<project name="AIBot" default="generate-protocols" basedir=".">
	<target name="init">
		<property name="outputDir" value="bin" />
		<property name="protocolsDir" value="protocols" />
		<property name="protocolsPackage" value="ovh.tgrhavoc.aibot.protocol" />
		<property name="deployJar" value="build/AIBot.jar" />
		<property name="libraryDir" value="lib" />
		<property name="tempDir" value="temp" />
		<property name="mainClass" value="ovh.tgrhavoc.aibot.wrapper.Main" />
		<property name="antContribPath" value="build/ant-contrib-1.0b3.jar" />
		<property name="resourceDir" value="resources"/>
	</target>

	<target name="clean" depends="init">
		<delete file="${deployJar}" failonerror="false" />
		<delete dir="${outputDir}" failonerror="false" />
		<delete dir="${tempDir}" failonerror="false" />
		<delete dir="${protocolsDir}" failonerror="false" />
	</target>

	<target name="compile" depends="clean">
		<mkdir dir="${outputDir}" />
		<javac encoding="UTF-8" destdir="${outputDir}" includeantruntime="false">
			<src path="src/" />
			<classpath>
				<fileset dir="${libraryDir}">
					<include name="**/*.jar" />
				</fileset>
			</classpath>
		</javac>
		<copy todir="${outputDir}">
			<fileset dir="src">
				<exclude name="**/*.java" />
			</fileset>
		</copy>
	</target>

	<target name="deploy" depends="compile">
		<mkdir dir="${tempDir}" />
		<unzip dest="${tempDir}">
			<patternset>
				<exclude name="META-INF/**" />
			</patternset>
			<fileset dir="${libraryDir}">
				<include name="**/*.jar" />
			</fileset>
		</unzip>
		<copy todir="${tempDir}">
			<fileset dir="${outputDir}">
				<include name="*" />
			</fileset>
			<fileset dir="${resourceDir}" />
		</copy>

		<echo message="Manifest-Version: 1.0${line.separator}Class-Path: .${line.separator}Main-Class: ${mainClass}" file="${outputDir}/manifest.txt" />
		<jar destfile="${deployJar}" basedir="${outputDir}" manifest="${outputDir}/manifest.txt" excludes="manifest.txt" />
		<jar destfile="${deployJar}" basedir="${tempDir}" includes="**/**" update="true" />
		<delete dir="${tempDir}" />
	</target>

	<target name="generate-protocols" depends="deploy">
		<taskdef resource="net/sf/antcontrib/antlib.xml">
			<classpath>
				<pathelement location="${antContribPath}" />
			</classpath>
		</taskdef>
		<mkdir dir="${protocolsDir}" />
		<javac encoding="UTF-8" destdir="${protocolsDir}" includeantruntime="false">
			<src path="protocols" />
			<classpath>
				<fileset file="${deployJar}" />
				<fileset dir="${libraryDir}">
					<include name="**/*.jar" />
				</fileset>
			</classpath>
		</javac>
		<propertyregex property="protocolsFolder" input="src.${protocolsPackage}" global="true" regexp="\." replace="/" />
		<pathconvert property="files" pathsep="${line.separator}">
			<map from="${protocolsFolder}{$file.separator}" to="" />
			<dirset dir="${protocolsFolder}">
				<include name="*" />
			</dirset>
		</pathconvert>
		<propertyregex property="protocolsFolder" override="true" input="${protocolsPackage}" global="true" regexp="\." replace="/" />
		<for list="${files}" delimiter="${line.separator}" param="file">
			<sequential>
				<local name="protocolVersion" />
				<basename property="protocolVersion" file="@{file}" />
				<mkdir dir="${protocolsDir}/META-INF/services" />
				<if>
				    <contains string="${protocolVersion}" substring="x" />
				    <then>
				    	<propertyregex property="protocolVersion" override="true" input="${protocolVersion}" regexp="v([0-9]+)" select="\1" />
				    	<echo message="${protocolsPackage}.v${protocolVersion}x.Protocol${protocolVersion}X$$Provider" file="${protocolsDir}/META-INF/services/${protocolsPackage}.ProtocolProvider" />
				    	<jar destfile="${protocolsDir}/v${protocolVersion}x.jar" update="false" basedir="${protocolsDir}" includes="${protocolsFolder}/v${protocolVersion}x/**/**,META-INF/**/**" />
				    </then>
				    <else>
				    	<propertyregex property="protocolVersion" override="true" input="${protocolVersion}" regexp="v([0-9]+)" select="\1" />
				    	<echo message="${protocolsPackage}.v${protocolVersion}.Protocol${protocolVersion}$$Provider" file="${protocolsDir}/META-INF/services/${protocolsPackage}.ProtocolProvider" />
				    	<jar destfile="${protocolsDir}/v${protocolVersion}.jar" update="false" basedir="${protocolsDir}" includes="${protocolsFolder}/v${protocolVersion}/**/**,META-INF/**/**" />
				    </else>
				</if>
				<delete dir="${protocolsDir}/META-INF" />
			</sequential>
		</for>
		<propertyregex property="protocolBase" input="${protocolsPackage}" regexp="^([a-zA-Z][a-zA-Z0-9]+).*" select="\1" />
		<delete dir="${protocolsDir}/${protocolBase}" />
	</target>
</project>
