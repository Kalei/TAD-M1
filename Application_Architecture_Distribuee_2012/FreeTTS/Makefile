
test:
	javac test.java
	java test

freetts:
	for i in `find com -name "*.java"`; do javac -cp "jsapi.jar:." "$$i"; done;
	for i in `find de -name "*.java"`; do javac -cp "jsapi.jar:." "$$i"; done;

clean:
	for i in `find . -name "*.class"`; do rm "$$i"; done;
