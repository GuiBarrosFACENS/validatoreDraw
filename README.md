import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

public class WhitelistValidation {

    private static final Map<String, Function<BilledInvoiceYK, Object>> fieldAccessors =
            new HashMap<>();

    static {
        fieldAccessors.put("message", BilledInvoiceYK::getMessage);
        fieldAccessors.put("function", BilledInvoiceYK::getFunction);
        fieldAccessors.put("paymentType", BilledInvoiceYK::getPaymentType);
        fieldAccessors.put("issueDate", BilledInvoiceYK::getIssueDate);
        fieldAccessors.put("paymentDate", BilledInvoiceYK::getPaymentDate);
        fieldAccessors.put("bankCode", BilledInvoiceYK::getBankCode);
        fieldAccessors.put("paymentChannel", BilledInvoiceYK::getPaymentChannel);
        fieldAccessors.put("paymentKind", BilledInvoiceYK::getPaymentKind);
        fieldAccessors.put("covenant", BilledInvoiceYK::getCovenant);
        fieldAccessors.put("typeOfPersonAgreement", BilledInvoiceYK::getTypeOfPersonAgreement);
        fieldAccessors.put("agreementDocument", BilledInvoiceYK::getAgreementDocument);
        fieldAccessors.put("clientNumber", BilledInvoiceYK::getClientNumber);
        fieldAccessors.put("txId", BilledInvoiceYK::getTxId);
        fieldAccessors.put("payerDocumentType", BilledInvoiceYK::getPayerDocumentType);
        fieldAccessors.put("payerDocumentNumber", BilledInvoiceYK::getPayerDocumentNumber);
        fieldAccessors.put("payerName", BilledInvoiceYK::getPayerName);
        fieldAccessors.put("dueDate", BilledInvoiceYK::getDueDate);
    }

    public static Map<String, String> validateFields(BilledInvoiceYK billedInvoiceYK) {
        Map<String, String> invalidFields = new HashMap<>();

        if (!billedInvoiceYK.isPayd()) {
            invalidFields.put("payedValue", "cannot be null or empty!");
        }

        for (Map.Entry<String, Function<BilledInvoiceYK, Object>> entry : fieldAccessors.entrySet()) {
            String fieldName = entry.getKey();
            Object fieldValue = entry.getValue().apply(billedInvoiceYK);

            if (isNullOrEmpty(fieldValue)) {
                invalidFields.put(fieldName, String.format("%s cannot be null or empty!", fieldName));
            }
        }

        return invalidFields;
    }

    private static boolean isNullOrEmpty(Object value) {
        if (value == null) {
            return true;
        }
        if (value instanceof String) {
            return ((String) value).trim().isEmpty();
        }
        if (value instanceof BigDecimal) {
            return ((BigDecimal) value).compareTo(BigDecimal.ZERO) <= 0;
        }
        return false;
    }
}



####### aaa

import java.util.Map;

public class ValidateBilledInvoiceYKCommand {

  public boolean execute(BilledInvoiceYK billedInvoiceYK) {
    Map<String, String> constrainsViolation = WhitelistValidation.validateFields(billedInvoiceYK);
    if (!constrainsViolation.isEmpty()) {
      throw new ValidationException(createMessage(constrainsViolation));
    }
    return true;
  }

  private String createMessage(Map<String, String> constrainsViolation) {
    StringBuilder message = new StringBuilder();
    constrainsViolation.forEach(
        (key, value) -> message.append(String.format("Field '%s' is value '%s' %n", key, value)));
    return message.toString();
  }
}


####### aaa


import java.lang.reflect.Field;
import java.math.BigDecimal;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class WhitelistValidation {

    private static final List<String> nullableFields = Arrays.asList(
        "participantCode",
        "finalBeneficiaryrDocumentType",
        "finalBeneficiaryDocumentNumber",
        "finalBeneficiaryName"
    );

    public static Map<String, String> validateFields(BilledInvoiceYK billedInvoiceYK) {
        Map<String, String> invalidFields = new HashMap<>();

        // Ensure billedInvoiceYK.isPayd() is true
        if (!billedInvoiceYK.isPayd()) {
            invalidFields.put("isPayd", "Billed invoice must be paid.");
        }

        // Validate all fields
        Field[] fields = BilledInvoiceYK.class.getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);
            try {
                Object value = field.get(billedInvoiceYK);
                String fieldName = field.getName();
                if (!nullableFields.contains(fieldName) && isNullOrEmpty(value)) {
                    invalidFields.put(fieldName, String.format("%s cannot be null or empty!", fieldName));
                }
            } catch (IllegalAccessException e) {
                e.printStackTrace();
                // Handle exception or rethrow
            }
        }

        return invalidFields;
    }

    private static boolean isNullOrEmpty(Object value) {
        if (value == null) {
            return true;
        }
        if (value instanceof String) {
            return ((String) value).trim().isEmpty();
        }
        if (value instanceof BigDecimal) {
            return ((BigDecimal) value).compareTo(BigDecimal.ZERO) <= 0;
        }
        return false;
    }
}

## aaa teste inverso

import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import java.math.BigDecimal;
import java.util.Map;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

public class WhitelistValidationTest {

    @Test
    public void testValidateFields_WithValidData() throws Exception {
        BilledInvoiceYK billedInvoiceYK = mock(BilledInvoiceYK.class);

        when(billedInvoiceYK.isPayd()).thenReturn(true);
        mockFieldValues(billedInvoiceYK, false);

        Map<String, String> result = WhitelistValidation.validateFields(billedInvoiceYK);

        assertTrue(result.isEmpty(), "No validation errors expected for valid data.");
    }

    @Test
    public void testValidateFields_WithInvalidData() throws Exception {
        BilledInvoiceYK billedInvoiceYK = mock(BilledInvoiceYK.class);

        when(billedInvoiceYK.isPayd()).thenReturn(false);
        mockFieldValues(billedInvoiceYK, true);

        Map<String, String> result = WhitelistValidation.validateFields(billedInvoiceYK);

        assertFalse(result.isEmpty(), "Validation errors expected for invalid data.");
        assertEquals("Billed invoice must be paid.", result.get("isPayd"));

        for (String key : WhitelistValidationTest.getAllFieldNames()) {
            if (!WhitelistValidation.nullableFields.contains(key)) {
                assertEquals(key + " cannot be null or empty!", result.get(key));
            }
        }
    }

    @Test
    public void testValidateFields_WithNullableFields() throws Exception {
        BilledInvoiceYK billedInvoiceYK = mock(BilledInvoiceYK.class);

        when(billedInvoiceYK.isPayd()).thenReturn(true);
        mockNullableFields(billedInvoiceYK);
        mockFieldValues(billedInvoiceYK, false);

        Map<String, String> result = WhitelistValidation.validateFields(billedInvoiceYK);

        assertTrue(result.isEmpty(), "No validation errors expected for nullable fields.");
    }

    private void mockFieldValues(BilledInvoiceYK billedInvoiceYK, boolean setNull) throws Exception {
        for (Field field : BilledInvoiceYK.class.getDeclaredFields()) {
            field.setAccessible(true);
            Object value = setNull ? null : getMockValueForField(field);
            Mockito.when(field.get(billedInvoiceYK)).thenReturn(value);
        }
    }

    private void mockNullableFields(BilledInvoiceYK billedInvoiceYK) throws Exception {
        for (String fieldName : WhitelistValidation.nullableFields) {
            Field field = BilledInvoiceYK.class.getDeclaredField(fieldName);
            field.setAccessible(true);
            Mockito.when(field.get(billedInvoiceYK)).thenReturn(null);
        }
    }

    private Object getMockValueForField(Field field) {
        Class<?> fieldType = field.getType();
        if (fieldType == String.class) {
            return "test";
        } else if (fieldType == BigDecimal.class) {
            return BigDecimal.ONE;
        }
        return null;
    }

    private static List<String> getAllFieldNames() {
        return Arrays.stream(BilledInvoiceYK.class.getDeclaredFields())
                .map(Field::getName)
                .collect(Collectors.toList());
    }
}
Explicação dos Testes
testValidateFields_WithValidData:

Cria um mock de BilledInvoiceYK com todos os campos preenchidos corretamente.
Define isPayd como true.
Verifica que o mapa result está vazio, indicando que não há erros de validação.
testValidateFields_WithInvalidData:

Cria um mock de BilledInvoiceYK com todos os campos obrigatórios nulos ou inválidos e isPayd como false.
Verifica que o mapa result não está vazio e contém as chaves apropriadas para os campos inválidos, incluindo a verificação de que o boleto deve estar pago.
testValidateFields_WithNullableFields:

Cria um mock de BilledInvoiceYK com campos opcionais nulos e outros campos obrigatórios preenchidos corretamente.
Define isPayd como true.
Verifica que o mapa result está vazio, indicando que não há erros de validação para os campos opcionais.



### teste do que está hoje 


import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

public class WhitelistValidationTest {

    @Test
    public void testValidateFields_AllValid() {
        BilledInvoiceYK billedInvoiceYK = mock(BilledInvoiceYK.class);
        
        when(billedInvoiceYK.isPayd()).thenReturn(true);
        mockFieldValues(billedInvoiceYK, false);

        Map<String, String> result = WhitelistValidation.validateFields(billedInvoiceYK);

        assertTrue(result.isEmpty(), "No validation errors expected for valid data.");
    }

    @Test
    public void testValidateFields_NotPayd() {
        BilledInvoiceYK billedInvoiceYK = mock(BilledInvoiceYK.class);
        
        when(billedInvoiceYK.isPayd()).thenReturn(false);
        mockFieldValues(billedInvoiceYK, false);

        Map<String, String> result = WhitelistValidation.validateFields(billedInvoiceYK);

        assertEquals(1, result.size(), "One validation error expected when invoice is not paid.");
        assertEquals("cannot be null or empty!", result.get("payedValue"));
    }

    @Test
    public void testValidateFields_FieldNullOrEmpty() {
        BilledInvoiceYK billedInvoiceYK = mock(BilledInvoiceYK.class);

        when(billedInvoiceYK.isPayd()).thenReturn(true);
        mockFieldValues(billedInvoiceYK, true);

        Map<String, String> result = WhitelistValidation.validateFields(billedInvoiceYK);

        assertFalse(result.isEmpty(), "Validation errors expected for null or empty fields.");
        for (String fieldName : WhitelistValidation.fieldAccessors.keySet()) {
            if (!fieldName.equals("payedValue")) {
                assertEquals(String.format("%s cannot be null or empty!", fieldName), result.get(fieldName));
            }
        }
    }

    private void mockFieldValues(BilledInvoiceYK billedInvoiceYK, boolean setNull) {
        for (Map.Entry<String, Function<BilledInvoiceYK, Object>> entry : WhitelistValidation.fieldAccessors.entrySet()) {
            String fieldName = entry.getKey();
            Object value = setNull ? null : getMockValueForField(entry.getValue());
            Mockito.when(entry.getValue().apply(billedInvoiceYK)).thenReturn(value);
        }
    }

    private Object getMockValueForField(Function<BilledInvoiceYK, Object> function) {
        Class<?> returnType = function.apply(null).getClass();
        if (returnType == String.class) {
            return "test";
        } else if (returnType == BigDecimal.class) {
            return BigDecimal.ONE;
        }
        return null;
    }
}
Explicação dos Testes
testValidateFields_AllValid:

Cria um mock de BilledInvoiceYK com todos os campos preenchidos corretamente.
Define isPayd como true.
Verifica que o mapa result está vazio, indicando que não há erros de validação.
testValidateFields_NotPayd:

Cria um mock de BilledInvoiceYK com todos os campos preenchidos corretamente, mas isPayd como false.
Verifica que o mapa result contém um erro para o campo payedValue, indicando que o boleto não está pago.
testValidateFields_FieldNullOrEmpty:

Cria um mock de BilledInvoiceYK com todos os campos obrigatórios nulos ou vazios e isPayd como true.
Verifica que o mapa result não está vazio e contém as chaves apropriadas para os campos inválidos.

