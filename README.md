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



-----

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





