# javac1

![image](https://user-images.githubusercontent.com/72151701/205998544-c38d1002-6d88-43b7-8afb-4325e19e1a19.png)


1 - colocar o driver na pasta lib do tomcat

2 - criar um dynamic web project

3 - new runtime > apache > apache tomcat v? > next

4 - selecionar o diretorio do tomcat > finish

5 - next > next > marcar o Generate web.xml > finish

6 - adicionar no server

7 - criar o servlet > desmarcar DoPost e DoGet e marcar Service > finish 


![image](https://user-images.githubusercontent.com/72151701/205998619-a563090c-737c-41bc-873c-c1270c609424.png)

public class Conexao {

	private final String url = "jdbc:oracle:thin:@oracle.fiap.com.br:1521:ORCL";
	private final String driver = "oracle.jdbc.driver.OracleDriver";
	private final String user = "";
	private final String password = "";
	private Connection conexao;
	
	public Connection conectar() {
		try {
			Class.forName(driver);
			conexao = DriverManager.getConnection(url, user, password);			
		}
		catch(ClassNotFoundException e) {
			System.out.println("Erro ao carregar o driver");
		}
		catch(SQLException e) {
			System.out.println("Erro ao estabelecer conexão com o banco de dados");
		}
		
		return conexao;
	}
	
	public void desconectar() {
		try {
			conexao.close();
		}
		catch(SQLException e) {
			System.out.println("Erro ao desconectar do banco de dados\n" + e);
		}
	}
}



![image](https://user-images.githubusercontent.com/72151701/205999175-fd1ddab0-afe1-4861-a4a9-e8acddcad2e2.png)

public class CadastroCaronaServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;
       
	/**
	 * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		
		CaronaDAO dao = new CaronaDAO();
		
		Integer usuarioId = Integer.parseInt(request.getParameter("usuario"));
		Integer motoristaId = Integer.parseInt(request.getParameter("motorista"));
		
		Usuario usuario = new Usuario();
		usuario.setId(usuarioId);
		
		Motorista motorista = new Motorista();
		motorista.setId(motoristaId);
		
		Carona carona = new Carona();
		carona.setUsuario(usuario);
		carona.setMotorista(motorista);
		
		dao.inserir(carona);
		
		// Redireciona para o index.jsp
		RequestDispatcher dispatcher = request.getRequestDispatcher("index.jsp");
		dispatcher.forward(request, response);
		
	}

}



![image](https://user-images.githubusercontent.com/72151701/205999753-ecd32db8-d549-48df-83a8-41128e640937.png)

public class CadastroMotoristaServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;
       
	/**
	 * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		
		MotoristaDAO dao = new MotoristaDAO();
		Motorista motorista = new Motorista();
		
		motorista.setNome(request.getParameter("nome"));
		motorista.setBairro(request.getParameter("bairro"));
		
		dao.inserir(motorista);
		
		// Redireciona para o index.jsp
		RequestDispatcher dispatcher = request.getRequestDispatcher("index.jsp");
		dispatcher.forward(request, response);
		
	}

}



![image](https://user-images.githubusercontent.com/72151701/205999873-01371f8b-6563-4101-a97f-b588ff09b999.png)

public class CadastroUsuarioServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;
       
	/**
	 * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		
		UsuarioDAO dao = new UsuarioDAO();
		Usuario usuario = new Usuario();
		
		usuario.setRm(Integer.parseInt(request.getParameter("rm")));
		usuario.setNome(request.getParameter("nome"));
		
		dao.inserir(usuario);
		
		// Redireciona para o index.jsp
		RequestDispatcher dispatcher = request.getRequestDispatcher("index.jsp");
		dispatcher.forward(request, response);
		
	}

}



![image](https://user-images.githubusercontent.com/72151701/206000134-be1c107b-0d7c-4a19-860e-27b1cdbca092.png)

public class CaronaDAO extends DAO {
	
	//Método para inserir uma carona na tabela java_carona
	public void inserir(Carona carona) {
		Conexao conexao = new Conexao();
		connection = conexao.conectar();
		sql = "INSERT INTO java_carona VALUES (carona_sequence.nextval, ?, ?)";
				
		try {
			ps = connection.prepareStatement(sql);
			ps.setInt(1, carona.getMotorista().getId());
			ps.setInt(2, carona.getUsuario().getId());
			ps.execute();
			ps.close();
			conexao.desconectar();
		}
		catch(SQLException e) {
			System.out.println("Erro ao inserir uma carona no BD\n" + e);
		}
				
	}
	
	//Método para listar as caronas da tabela java_carona 
	//Trazendo informações dos usuários e motoristas provenientes de outras tabelas
	
	public List<Carona> listar() {
		List<Carona> lista = new ArrayList<Carona>();
		Carona carona;
		Usuario usuario;
		Motorista motorista;
		Conexao conexao = new Conexao();
		connection = conexao.conectar();
		sql = "SELECT U.nome as Unome, U.rm as Urm, M.nome as Mnome, M.bairro as Mbairro, "
				+ "C.id, C.id_usuario, C.id_motorista FROM java_carona C "
				+ "INNER JOIN java_usuario U ON U.id = C.id_usuario "
				+ "INNER JOIN java_motorista M ON M.id = C.id_motorista ORDER BY id";
		
		try {
			ps = connection.prepareStatement(sql);
			rs = ps.executeQuery();
			while(rs.next()) {
				usuario = new Usuario();
				usuario.setNome(rs.getString("Unome"));
				usuario.setRm(rs.getInt("Urm"));
				
				motorista = new Motorista();
				motorista.setNome(rs.getString("Mnome"));
				motorista.setBairro(rs.getString("Mbairro"));
				
				carona = new Carona();
				carona.setId(rs.getInt("id"));
				carona.setUsuario(usuario);
				carona.setMotorista(motorista);
				lista.add(carona);
			}
			ps.close();
			conexao.desconectar();
		}
		catch(SQLException e) {
			System.out.println("Erro ao listar as caronas do BD\n" + e);
		}
		return lista;
	}
	
}



![image](https://user-images.githubusercontent.com/72151701/206000318-0e6559c2-54b9-4e4d-9644-382fe9a7a148.png)

public class DAO {

	protected Connection connection;
	protected PreparedStatement ps;
	protected ResultSet rs;
	protected String sql;
  
}



![image](https://user-images.githubusercontent.com/72151701/206000398-f3e2f6cf-354f-4efd-b8e3-6befb21faa3d.png)

public class MotoristaDAO extends DAO {

	//Método para inserir um motorista na tabela java_motorista
		public void inserir(Motorista motorista) {
			Conexao conexao = new Conexao();
			connection = conexao.conectar();
			sql = "INSERT INTO java_motorista VALUES (motorista_sequence.nextval, ?, ?)";
			
			try {
				ps = connection.prepareStatement(sql);
				ps.setString(1, motorista.getNome());
				ps.setString(2, motorista.getBairro());
				ps.execute();
				ps.close();
				conexao.desconectar();
			}
			catch(SQLException e) {
				System.out.println("Erro ao inserir um motorista no BD\n" + e);
			}
			
		}
		
		//Método para listar os motoristas da tabela java_motorista
		public List<Motorista> listar() {
			List<Motorista> lista = new ArrayList<Motorista>();
			Motorista motorista;
			Conexao conexao = new Conexao();
			connection = conexao.conectar();
			sql = "SELECT * FROM JAVA_MOTORISTA";
			
			try {
				ps = connection.prepareStatement(sql);
				rs = ps.executeQuery();
				while(rs.next()) {
					motorista = new Motorista();
					motorista.setId(rs.getInt("id"));
					motorista.setNome(rs.getString("nome"));
					motorista.setBairro(rs.getString("bairro"));
					lista.add(motorista);
				}
				ps.close();
				conexao.desconectar();
			}
			catch(SQLException e) {
				System.out.println("Erro ao listar os motoristas do BD\n" + e);
			}
			return lista;
		}
	
}



![image](https://user-images.githubusercontent.com/72151701/206000603-9f669354-ea0e-4d24-9f15-47beba7cb9e3.png)

public class UsuarioDAO extends DAO{
	
	//Método para inserir um usuário na tabela java_usuario
	public void inserir(Usuario usuario) {
		Conexao conexao = new Conexao();
		connection = conexao.conectar();
		sql = "INSERT INTO java_usuario VALUES (usuario_sequence.nextval, ?, ?)";
		
		try {
			ps = connection.prepareStatement(sql);
			ps.setInt(1, usuario.getRm());
			ps.setString(2, usuario.getNome());
			ps.execute();
			ps.close();
			conexao.desconectar();
		}
		catch(SQLException e) {
			System.out.println("Erro ao inserir um usuario no BD\n" + e);
		}
		
	}
	
	//Método para listar os usuários da tabela java_usuario
	public List<Usuario> listar() {
		List<Usuario> lista = new ArrayList<Usuario>();
		Usuario usuario;
		Conexao conexao = new Conexao();
		connection = conexao.conectar();
		sql = "SELECT * FROM JAVA_USUARIO";
		
		try {
			ps = connection.prepareStatement(sql);
			rs = ps.executeQuery();
			while(rs.next()) {
				usuario = new Usuario();
				usuario.setId(rs.getInt("id"));
				usuario.setRm(rs.getInt("rm"));
				usuario.setNome(rs.getString("nome"));
				lista.add(usuario);
			}
			ps.close();
			conexao.desconectar();
		}
		catch(SQLException e) {
			System.out.println("Erro ao listar os usuarios do BD\n" + e);
		}
		return lista;
	}
	
}



![image](https://user-images.githubusercontent.com/72151701/206000790-6060097d-c0ce-4815-99ee-3f1386fdefe1.png)

public class Carona {

	private Integer id;
	private Usuario usuario;
	private Motorista motorista;
	
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public Usuario getUsuario() {
		return usuario;
	}
	public void setUsuario(Usuario usuario) {
		this.usuario = usuario;
	}
	public Motorista getMotorista() {
		return motorista;
	}
	public void setMotorista(Motorista motorista) {
		this.motorista = motorista;
	}
	
}



![image](https://user-images.githubusercontent.com/72151701/206000861-31779746-bd85-4f57-a89e-c16c21ab813c.png)

public class Motorista {

	private Integer id;
	private String nome;
	private String bairro;
	
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getNome() {
		return nome;
	}
	public void setNome(String nome) {
		this.nome = nome;
	}
	public String getBairro() {
		return bairro;
	}
	public void setBairro(String bairro) {
		this.bairro = bairro;
	}
	
}



![image](https://user-images.githubusercontent.com/72151701/206000955-8b92290d-2e2b-4017-96a6-778f312ebae2.png)

public class Usuario {

	private Integer id;
	private Integer rm;
	private String nome;
	
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public Integer getRm() {
		return rm;
	}
	public void setRm(Integer rm) {
		this.rm = rm;
	}
	public String getNome() {
		return nome;
	}
	public void setNome(String nome) {
		this.nome = nome;
	}
	
}



![image](https://user-images.githubusercontent.com/72151701/206042646-e38a3462-74d3-448c-88e8-96b40bf0953e.png)

cadastrarcarona {

	<%List<Usuario> listaUsuarios = new UsuarioDAO().listar();%>
	<%List<Motorista> listaMotoristas = new MotoristaDAO().listar();%>
	<form method="post" action="cadastrarcarona">
		<h1>Cadastro de Caronas</h1>
		<p>
			<select name="usuario">
				<option value="" disabled selected>Selecione um usuário</option>
				<% for(Usuario usuario : listaUsuarios) { %>
					<option value="<%=usuario.getId()%>">
					<%= " Usuário: " + usuario.getNome() + " (RM" + usuario.getRm() + ") " %>
					</option>
				<% } %>
			</select>
		</p>
		<p>
			<select name="motorista">
				<option value="" disabled selected>Selecione um motorista</option>
				<% for(Motorista motorista : listaMotoristas) { %>
					<option value="<%=motorista.getId()%>">
					<%= " Motorista: " + motorista.getNome() + " (Bairro: " + motorista.getBairro() + ") " %>
					</option>
				<% } %>
			</select>
		</p>
		<p>
			<input type="submit" value="Cadastrar Carona"/>
		</p>
	</form>
	
}



![image](https://user-images.githubusercontent.com/72151701/206043349-bc864cc4-e6d6-4291-b458-8a1237347395.png)

cadastrarmotorista {

	<form method="post" action="cadastrarmotorista">
		<h1>Cadastro de Motoristas</h1>
		<p>
			<label for="nome">Nome</label>
			<input id="nome" name="nome" required="required" 
			type="text" placeholder="Qual o seu nome?"/>
		</p>
		<p>
			<label for="bairro">Bairro</label>
			<input id="bairro" name="bairro" required="required" 
			type="text" placeholder="Vila Mariana"/>
		</p>
		<p>
			<input type="submit" value="Cadastrar Motorista"/>
		</p>
	</form>
	
}



![image](https://user-images.githubusercontent.com/72151701/206043510-bc085fe2-5021-42c9-9d17-e5c933c0bfcc.png)

cadastrarusuario {

	<form method="post" action="cadastrarusuario">
		<h1>Cadastro de Usuários</h1>
		<p>
			<label for="rm">RM</label>
			<input id="rm" name="rm" required="required" 
			type="number" placeholder="12345"/>
		</p>
		<p>
			<label for="nome">Nome</label>
			<input id="nome" name="nome" required="required" 
			type="text" placeholder="Qual o seu nome?"/>
		</p>
		<p>
			<input type="submit" value="Cadastrar Usuario"/>
		</p>
	</form>
	
}	


![image](https://user-images.githubusercontent.com/72151701/206043621-5fea089a-01ba-436b-8122-09f5f02e4e98.png)

index {

	<form action="cadastrarusuario.jsp">
		<button type="submit">Cadastrar Usuário</button>
	</form>
	
	<form action="cadastrarmotorista.jsp">
		<button type="submit">Cadastrar Motorista</button>
	</form>
	
	<form action="cadastrarcarona.jsp">
		<button type="submit">Cadastrar Carona</button>
	</form>
	
	<form action="listarcaronas.jsp">
		<button type="submit">Listar Caronas</button>
	</form>
	
}


![image](https://user-images.githubusercontent.com/72151701/206044015-8d650ea2-02f5-4f6d-8ae8-72cbdef506f8.png)


listarcaronas {

	<form action="cadastrarusuario.jsp">
		<button type="submit">Cadastrar Usuário</button>
	</form>
	
	<form action="cadastrarmotorista.jsp">
		<button type="submit">Cadastrar Motorista</button>
	</form>
	
	<form action="cadastrarcarona.jsp">
		<button type="submit">Cadastrar Carona</button>
	</form>
	
	<form action="listarcaronas.jsp">
		<button type="submit">Listar Caronas</button>
	</form>
	
}









