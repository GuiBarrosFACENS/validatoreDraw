# validatoreDraw


1. Defina um grupo de validação
Crie uma interface de marcação para o grupo de validação que você deseja aplicar. Por exemplo:

java
Copiar código
package com.example.demo.validation;

public interface PaidBoletoValidationGroup {
}
2. Adicione anotações de validação específicas para o grupo no modelo Boleto
Adicione as anotações de validação aos campos relevantes no modelo Boleto e associe-os ao grupo de validação recém-criado:

java
Copiar código
package com.example.demo.domain;

import com.example.demo.validation.PaidBoletoValidationGroup;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.validation.constraints.*;
import java.math.BigDecimal;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Boleto {
    private String codigoDoBoleto;

    private String dataDeVencimento;

    @NotNull(message = "Valor do Boleto não pode ser nulo", groups = PaidBoletoValidationGroup.class)
    @DecimalMin(value = "0.01", message = "Valor do Boleto deve ser maior que zero", groups = PaidBoletoValidationGroup.class)
    private BigDecimal valorDoBoleto;

    @NotBlank(message = "Nome do Pagador não pode ser nulo ou vazio", groups = PaidBoletoValidationGroup.class)
    private String nomeDoPagador;

    @NotBlank(message = "CPF do Pagador não pode ser nulo ou vazio", groups = PaidBoletoValidationGroup.class)
    private String cpfDoPagador;

    private String bancoEmissor;

    private String linhaDigitavel;

    private boolean pago;
}
3. Atualize o serviço BoletoService para usar a validação condicional
No serviço, valide o objeto Boleto apenas se ele estiver marcado como pago:

java
Copiar código
package com.example.demo.service;

import com.example.demo.domain.Boleto;
import com.example.demo.validation.PaidBoletoValidationGroup;
import org.springframework.stereotype.Service;
import org.springframework.validation.annotation.Validated;

import javax.validation.Valid;
import javax.validation.Validator;

@Service
public class BoletoService {
    private final Validator validator;

    public BoletoService(Validator validator) {
        this.validator = validator;
    }

    public void validarBoleto(@Valid Boleto boleto) {
        if (boleto.isPago()) {
            var violations = validator.validate(boleto, PaidBoletoValidationGroup.class);
            if (!violations.isEmpty()) {
                throw new ValidationException("Todos os campos obrigatórios devem ser preenchidos quando o boleto está pago.");
            }
        }
    }
}
4. Atualize o Controller para usar @Validated
Garanta que seu controller utilize a anotação @Validated para ativar a validação em nível de método:

java
Copiar código
package com.example.demo.api;

import com.example.demo.domain.Boleto;
import com.example.demo.service.BoletoService;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;
import javax.validation.ValidationException;

@RestController
@Validated
public class BoletoApiController {

    private final BoletoService boletoService;

    public BoletoApiController(BoletoService boletoService) {
        this.boletoService = boletoService;
    }

    public ResponseEntity<String> processBoleto(@Valid @RequestBody Boleto boleto) {
        boletoService.validarBoleto(boleto);
        return ResponseEntity.ok("Dados válidos");
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<String> handleValidationException(ValidationException ex) {
        return ResponseEntity.badRequest().body(ex.getMessage());
    }
}
5. Atualize o GlobalExceptionHandler (opcional)
Se desejar, você pode manter o GlobalExceptionHandler para capturar exceções de validação personalizadas:

java
Copiar código
package com.example.demo.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<String> handleValidationException(ValidationException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(ex.getMessage());
    }
}
Resumo
Crie um grupo de validação: Use uma interface para definir o grupo.
Adicione anotações ao modelo: Especifique as validações apenas para os campos relevantes usando o grupo.
Valide condicionalmente: No serviço, valide apenas se o campo pago estiver definido.
Atualize o controller: Use @Validated para aplicar a validação no nível do método.
Mantenha o ExceptionHandler: Opcionalmente, capture exceções de validação personalizadas globalmente.
Dessa forma, apenas os campos especificados são validados quando pago é verdadeiro, enquanto os demais campos podem ser nulos.
