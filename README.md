import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.*;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.springframework.kafka.core.KafkaTemplate;

public class KafkaProviderTest {

    @Mock
    private KafkaTemplate<String, String> kafkaTemplate;

    @InjectMocks
    private KafkaProvider kafkaProvider;

    private static final String TOPIC_KAFKA = "topic_kafka";
    private static final String JSON_EXPECTED = "{\"key\":\"value\"}";

    @Test
    void sendTopicInterruptedException() throws InterruptedException, ExecutionException {
        when(kafkaTemplate.send(TOPIC_KAFKA, JSON_EXPECTED)).thenThrow(new RuntimeException(new InterruptedException()));

        BilledInvoicePlard invoicePlard = new BilledInvoicePlard(0L, null, null, null);

        CompletableFuture<Void> future = kafkaProvider.sendTopic(invoicePlard);
        ExecutionException exception = assertThrows(ExecutionException.class, future::get);

        assertTrue(exception.getCause() instanceof ErrorSendTopickAfkaException);
        verify(kafkaTemplate, times(1)).send(TOPIC_KAFKA, JSON_EXPECTED);
    }

    @Test
    void sendTopicKafkaException() throws InterruptedException, ExecutionException {
        CompletableFuture<CompletableFuture.SendResult<String, String>> result = mock(CompletableFuture.class);
        when(kafkaTemplate.send(TOPIC_KAFKA, JSON_EXPECTED)).thenReturn(result);
        when(result.exceptionally(any())).thenAnswer(invocation -> {
            throw new KafkaException("Error to send kafka message");
        });

        BilledInvoicePlard invoicePlard = new BilledInvoicePlard(0L, null, null, null);

        CompletableFuture<Void> future = kafkaProvider.sendTopic(invoicePlard);
        ExecutionException exception = assertThrows(ExecutionException.class, future::get);

        assertTrue(exception.getCause() instanceof ErrorSendTopickAfkaException);
        verify(kafkaTemplate, times(1)).send(TOPIC_KAFKA, JSON_EXPECTED);
    }
}
