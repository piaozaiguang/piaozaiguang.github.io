_Suppose you have a [Slack](https://slack.com) account_

### Preview
![slack_window](https://cloud.githubusercontent.com/assets/6111081/25732618/3eae5c2c-3184-11e7-95a4-e802ac3a0284.png)

### Integration
#### Add integration
![slack_add_apps](https://cloud.githubusercontent.com/assets/6111081/25732637/72ffa7d8-3184-11e7-9678-732e8fbeb9f5.png)

#### Search & add integration
![search_app](https://cloud.githubusercontent.com/assets/6111081/25732789/bb1680cc-3185-11e7-8c6f-e03e5cef4192.png)

#### Add configuration
![add_configuration](https://cloud.githubusercontent.com/assets/6111081/25732655/b8559e5a-3184-11e7-8dfa-7c6e8855dd26.png)

#### Select channel
![select_channel](https://cloud.githubusercontent.com/assets/6111081/25732798/d9659b62-3185-11e7-9ebc-2023947028d4.png)

#### Copy webhook URL & save settings
![webhook_url](https://cloud.githubusercontent.com/assets/6111081/25732816/ffde8d4e-3185-11e7-8f78-553ac1518ec3.png)

#### logback.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<include resource="org/springframework/boot/logging/logback/base.xml"/>
	<appender name="SLACK" class="io.github.piaozaiguang.logback.LogbackSlackAppender">
        <level>ERROR</level>
        <endpoint>https://hooks.slack.com/services/XXXXXXXXX/XXXXXXXXX/XxxxxXXxXXXxxxxXXXXXxxxX</endpoint>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[${HOSTNAME}][Admin RELEASE] %date %-5level - %logger{0} - %message%n</pattern>
        </layout>
	</appender>

	<!-- Root Logger -->
	<root level="ALL">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
		<appender-ref ref="SLACK" />
	</root>
</configuration>
```
#### LogbackSlackAppender.java
```java
package io.github.piaozaiguang.logback;

import org.apache.commons.lang.StringUtils;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.RestTemplate;

import ch.qos.logback.classic.Level;
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.Layout;
import ch.qos.logback.core.UnsynchronizedAppenderBase;
import ch.qos.logback.core.status.ErrorStatus;
import lombok.Getter;
import lombok.Setter;
import lombok.Value;
import lombok.extern.slf4j.Slf4j;

/**
 * Created by piaozaiguang on 2017/4/8.
 */
@Getter
@Setter
@Slf4j
public class LogbackSlackAppender extends UnsynchronizedAppenderBase<ILoggingEvent> {
    private static final String SLACK_LOGGING = "slack logging response: ";
    private static final String SLACK_LOGGING_RESPONSE = SLACK_LOGGING + "{}";
    private String endpoint;
    private Level level;
    private Layout<ILoggingEvent> layout;

    @Override
    public void start() {
        int errors = 0;
        if (level == null) {
            addStatus(new ErrorStatus("No level set for the appender named \"" + name + "\".", this));
            errors++;
        }
        if (endpoint == null) {
            addStatus(new ErrorStatus("No endpoint set for the appender named \"" + name + "\".", this));
            errors++;
        }
        if (layout == null) {
            addStatus(new ErrorStatus("No layout set for the appender named \"" + name + "\".", this));
            errors++;
        }
        if (errors == 0) {
            super.start();
        }
    };

    @Override
    protected void append(ILoggingEvent evt) {
        if (!isStarted()) {
            return;
        }

        if (!StringUtils.contains(evt.getMessage(), SLACK_LOGGING)) {
            if (evt.getLevel().isGreaterOrEqual(level)) {
                Message message = new Message(layout.doLayout(evt));
                RestTemplate rt = new RestTemplate();
                ResponseEntity<String> response = postMessage(message, rt);
                log.debug(SLACK_LOGGING_RESPONSE, response);
            }
        }
    }

    protected ResponseEntity<String> postMessage(Message message, RestTemplate rt) {
        ResponseEntity<String> response = rt.postForEntity(endpoint, message, String.class);
        return response;
    }

    @Value
    static class Message {
        private String text;
    }
}
```
