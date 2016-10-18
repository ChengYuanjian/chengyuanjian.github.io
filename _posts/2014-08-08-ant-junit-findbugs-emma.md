---
layout:         post
title:         利用ANT生成JUnit、Findbugs、Emma报告
description: 使用ANT脚本来运行Junit、Findbugs、Emma，并产出报告 
keywords: ANT,JUNIT,FINDBUGS,EMMA
category: ANT
tags: [ANT,JUNIT,FINDBUGS,EMMA]
---

* EMMA是一个用于检测和报告JAVA代码覆盖率的开源工具。它不但能很好的用于小型项目，很方便得得出覆盖率报告，而且适用于大型企业级别的项目。

* FindBugs是一个静态分析工具，它检查类或者JAR文件，将字节码与一组缺陷模式进行对比以发现可能的问题。有了静态分析工具，就可以在不实际运行程序的情况对软件进行分析。

* Ant是一种基于Java的build工具。理论上来说，它有些类似于（Unix）C中的make ，但没有make的缺陷。

* JUnit是一个开放源代码的Java测试框架，用于编写和运行可重复的测试。

本文主要介绍如何合理地利用这四者来生成单元报告、FindBugs报告和代码覆盖率报告。

<!-- more -->

###准备工作

首先自然是下载相关的jar包，下面列出所必需的jar包：
<pre><code>
asm-3.1.jar
asm-analysis-3.1.jar
asm-commons-3.1.jar
asm-tree-3.1.jar
asm-util-3.1.jar
asm-xml-3.1.jar
bcel.jar
dom4j-1.6.1.jar
emma.jar
emma_ant.jar
findbugs-ant.jar
findbugs.jar
jaxen-1.1.1.jar
jFormatString.jar
jsr305.jar
</code></pre>

###编写ANT脚本

下面列出完整的脚本示例，包括了编译、打包、运行JUnit并生成报表、生成Findbugs报表和Emma代码覆盖率报表：

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>

<project name="MyDemo" default="build" basedir=".">
	<property name="src.dir" value="src" />
	<property name="testsrc.dir" value="testsrc" />
	<property name="classes.dir" value="bin" />
	<property name="zip.name" value="MyDemo.zip" />

	<property name="build.dir" value="build" />
	<property name="junit.report.dir" value="report/junit" />
	<property name="findbugs.report.dir" value="report/findbugs" />
	<property name="coverage.report.dir" value="report/coverage" />
	<property name="bin.instrument.dir" location="report/coverage/instrbin" />
	<property name="emma.enabled" value="true" />


	<path id="jar.classpath">
		<fileset dir="lib">
			<include name="**.jar" />
		</fileset>
	</path>

	<path id="emma.lib">
		<pathelement location="lib/emma.jar" />
		<pathelement location="lib/emma_ant.jar" />
	</path>

	<target name="clean">
		<delete dir="${build.dir}" />
		<delete dir="${findbugs.report.dir}" />
		<delete dir="${junit.report.dir}" />
		<delete dir="${coverage.report.dir}" />
		<delete dir="${bin.instrument.dir}" />
	</target>

	<target name="prepare" depends="clean">
		<mkdir dir="${build.dir}" />
		<mkdir dir="${findbugs.report.dir}" />
		<mkdir dir="${junit.report.dir}" />
		<mkdir dir="${coverage.report.dir}" />
		<mkdir dir="${bin.instrument.dir}" />
	</target>

	<target name="compile" depends="prepare">
		<javac srcdir="${src.dir}" destdir="${classes.dir}" debug="true" deprecation="true">
			<classpath refid="jar.classpath">
			</classpath>
		</javac>
	</target>

	<target name="testcompile" depends="prepare">
		<javac srcdir="${testsrc.dir}" destdir="${classes.dir}" debug="true" deprecation="true">
			<classpath refid="jar.classpath">
			</classpath>
		</javac>
	</target>

	<!-- build zip file -->
	<target name="build" depends="compile">
		<zip destfile="${build.dir}/${zip.name}">
			<fileset dir=".">
				<include name="bin/**" />
				<include name="lib/**" />
				<exclude name="build.xml" />
			</fileset>
		</zip>
	</target>

	<target name="allcompile" depends="compile,testcompile">
	</target>

	<!-- run junit case -->
	<target name="junit" depends="allcompile">
		<junit printsummary="on" fork="true" showoutput="true">
			<classpath>
				<fileset dir="lib" includes="**/*.jar" />
				<pathelement path="${classes.dir}" />
			</classpath>
			<formatter type="xml" />
			<batchtest todir="${junit.report.dir}">
				<fileset dir="${classes.dir}">
					<include name="**/*TestAll*.*" />
				</fileset>
			</batchtest>
		</junit>
	</target>

	<!-- generate junit report -->
	<target name="junit-report" depends="junit">
		<junitreport todir="${junit.report.dir}">
			<fileset dir="${junit.report.dir}">
				<include name="TEST-*.xml" />
			</fileset>
			<report format="frames" todir="${junit.report.dir}" />
		</junitreport>
	</target>

	<taskdef name="findbugs" classname="edu.umd.cs.findbugs.anttask.FindBugsTask" classpathref="jar.classpath" />

	<!-- generate findbugs report -->
	<target name="findbug" depends="compile">
		<findbugs home="lib" output="html" outputFile="${findbugs.report.dir}/findbugs.html">
			<class location="${classes.dir}" />
			<sourcePath path="${src.dir}" />
		</findbugs>
	</target>

	<taskdef resource="emma_ant.properties" classpathref="jar.classpath" />

	<path id="classpath.main">
		<pathelement location="${classes.dir}" />
	</path>

	<target name="instrument" depends="allcompile">
		<emma enabled="${emma.enabled}">
			<instr instrpathref="classpath.main" destdir="${bin.instrument.dir}" metadatafile="${coverage.report.dir}/metadata.emma" merge="true">
			</instr>
		</emma>
		<copy todir="${bin.instrument.dir}">
			<fileset dir="bin">
				<exclude name="**/*.java" />
			</fileset>
		</copy>
	</target>

	<target name="run-junit" depends="instrument">
		<junit fork="true" forkmode="once" printsummary="withOutAndErr" errorproperty="test.error" showoutput="on">
			<jvmarg value="-Demma.coverage.out.file=${coverage.report.dir}/metadata.emma" />
			<jvmarg value="-Demma.coverage.out.merge=true" />
			<classpath location="${bin.instrument.dir}" />
			<classpath location="${classes.dir}" />
			<classpath refid="emma.lib" />
			<formatter type="xml" />
			<batchtest todir="${junit.report.dir}" haltonfailure="no">
				<fileset dir="${classes.dir}">
					<include name="**/*TestAll*.class" />
				</fileset>
			</batchtest>
		</junit>
	</target>

	<target name="gen-report-coverage" depends="run-junit">
		<emma enabled="${emma.enabled}">
			<report sourcepath="${src.dir}" sort="+block,+name,+method,+class" metrics="method:70,block:80,line:80,class:100">
				<fileset dir="${coverage.report.dir}">
					<include name="*.emma" />
				</fileset>
				<html outfile="${coverage.report.dir}/coverage.html" depth="method" columns="name,class,method,block,line" />
			</report>
		</emma>
	</target>

</project>
{% endhighlight %}
