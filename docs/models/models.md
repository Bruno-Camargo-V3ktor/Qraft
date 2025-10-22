User <> Organization: Uma relação "muitos-para-muitos". Um usuário pode estar em várias organizações (ex: a empresa dele, um projeto freelancer), e uma organização tem vários usuários. A tabela UserOrganization (tabela-pivô) define quem pertence a quem e qual o seu role (papel).

Organization -> QrCode: Cada QrCode pertence a uma Organization. Isso é essencial para o faturamento e permissões no futuro.

User -> QrCode: Guardamos o owner_id para saber qual usuário específico criou o QR Code.

QrCode -> Scan: Cada QrCode pode ter milhões de Scans. Esta é a base da sua tela de analytics. O qr_code_id na tabela Scan é a foreign key que faz essa ligação.
