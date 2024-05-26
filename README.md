
domain.entity.BilledInvoiceYK;
validation.WhitelistValidation;
infra.exception.ValidationException;

import java.util.HashMap;
import java.util.Map;

public class ValidateBilledInvoiceYKCommand {

    public boolean execute(BilledInvoiceYK billedInvoiceYK) {

        WhitelistValidation whitelistValidation = null;

        Map<String, String> constrainsViolation = new HashMap<>();

        if (billedInvoiceYK.isPayd()) {
            whitelistValidation.validateFields(billedInvoiceYK);
        } else {
            throw new ValidationException(createMessage(constrainsViolation));
        }
    }

    private String createMessage(Map<String, String> constrainsViolation) {
        StringBuilder message = new StringBuilder();
        constrainsViolation
                .keySet()
                .forEach(
                        key -> {
                            String fieldMessageError =
                                    String.format("Field '%s' is value '%s' %n", key, constrainsViolation.get(key));
                            message.append(fieldMessageError);
                        });
        return message.toString();
    }

}



import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

public class WhitelistValidation {

    private static final Map<String, Function<BilledInvoiceYK, Object>> fieldAccessors = new HashMap<>();

    static {
        fieldAccessors.put("participantCode", BilledInvoiceYK::getParticipantCode);
        fieldAccessors.put("finalBeneficiaryrDocumentType", BilledInvoiceYK::getFinalBeneficiaryrDocumentType);
        fieldAccessors.put("finalBeneficiaryDocumentNumber", BilledInvoiceYK::getFinalBeneficiaryDocumentNumber);
        fieldAccessors.put("finalBeneficiaryName", BilledInvoiceYK::getFinalBeneficiaryName);
    }

    public static boolean validateFields(BilledInvoiceYK billedInvoiceYK){
        for(Map.Entry<String, Function<BilledInvoiceYK, Object>> entry : fieldAccessors.entrySet()){
            String fieldName = entry.getKey();
            Object fieldValue = entry.getValue().apply(billedInvoiceYK);

            if (!isNullOrEmpty(fieldValue)) {
                throw new InvoicePaymentValidationException(String.format("%s não pode ser nulo ou vazio.", capitalize(fieldName)));
            }
        }
        return true;
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
