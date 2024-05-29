import com.fasterxml.jackson.databind.ObjectMapper;
import com.santander.mpa.domain.entity.BilledInvoiceYK;
import org.apache.commons.io.FileUtils;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;
import org.mockito.junit.jupiter.MockitoExtension;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.math.BigDecimal;
import java.util.Map;
import java.util.function.Function;

import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.util.ResourceUtils;

import java.util.Properties;

@ExtendWith(MockitoExtension.class)
@ActiveProfiles("teste")
class WhitelistValidationTest {

    @InjectMocks
    private WhitelistValidation whitelistValidation;

    @Mock
    private BilledInvoiceYK billedInvoiceYK;

    @BeforeEach
    void setUp() throws IOException {
        ObjectMapper objectMapper = new ObjectMapper();
        billedInvoiceYK = objectMapper.readValue(new File("src/test/resources/json/billedInvoiceYK.json"), BilledInvoiceYK.class);
    }


    @Test
    public void testValidateFields_AllValid() {
        when(billedInvoiceYK.isPayd()).thenReturn(true);
        Map<String, String> invalidFields = whitelistValidation.validateFields(billedInvoiceYK);
        assertTrue(invalidFields.isEmpty(), "cannot be null or empty!");
    }


}
