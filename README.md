# Caja-con-traslaci-nes-y-rotaci-nes-con-gui-
Codigo utilizando traslaciónes aplicadas a una arista del cubo en los 3 ejes, para luego aplicar las rotaciónes utilizando las cajas del gui 
classdef app1 < matlab.apps.AppBase

    % Properties that correspond to app components
    properties (Access = public)
        UIFigure          matlab.ui.Figure
        botonButton       matlab.ui.control.Button
        RotacionLabel     matlab.ui.control.Label
        TraslacionLabel   matlab.ui.control.Label
        RzEditField       matlab.ui.control.NumericEditField
        RzEditFieldLabel  matlab.ui.control.Label
        RyEditField       matlab.ui.control.NumericEditField
        RyEditFieldLabel  matlab.ui.control.Label
        RxEditField       matlab.ui.control.NumericEditField
        RxEditFieldLabel  matlab.ui.control.Label
        TzEditField       matlab.ui.control.NumericEditField
        TzEditFieldLabel  matlab.ui.control.Label
        TyEditField       matlab.ui.control.NumericEditField
        TyEditFieldLabel  matlab.ui.control.Label
        TxEditField       matlab.ui.control.NumericEditField
        TxEditFieldLabel  matlab.ui.control.Label
        UIAxes            matlab.ui.control.UIAxes
    end

    % Callbacks that handle component events
    methods (Access = private)

        % Button pushed function: botonButton
        function botonButtonPushed(app, event)
            % Button pushed function: botonButton

    % 1. Limpiar el plano y configurar los ejes
    cla(app.UIAxes);
    hold(app.UIAxes, 'on');
    grid(app.UIAxes, 'on');
    axis(app.UIAxes, 'equal');
    view(app.UIAxes, [45, 30]);

    % Pintar ejes de referencia (X: Azul, Y: Rojo, Z: Verde)
    line(app.UIAxes, [0 10], [0 0], [0 0], 'Color', 'blue', 'LineWidth', 2);
    line(app.UIAxes, [0 0], [0 10], [0 0], 'Color', 'red', 'LineWidth', 2);
    line(app.UIAxes, [0 0], [0 0], [0 10], 'Color', 'green', 'LineWidth', 2);

    % 2. Dimensiones estáticas por defecto del cubo (3x3x3)
    ancho = 3; largo = 3; alto = 3;

    % Dibujar cubo original estático en negro (z = 0 y z = alto)
    line(app.UIAxes, [0 ancho],[0 0],[0 0],'Color','black','LineWidth',1);
    line(app.UIAxes, [ancho ancho],[0 largo],[0 0],'Color','black','LineWidth',1);
    line(app.UIAxes, [ancho 0],[largo largo],[0 0],'Color','black','LineWidth',1);
    line(app.UIAxes, [0 0],[largo 0],[0 0],'Color','black','LineWidth',1);

    line(app.UIAxes, [0 0],[0 0],[0 alto],'Color','black','LineWidth',1);
    line(app.UIAxes, [ancho ancho],[0 0],[0 alto],'Color','black','LineWidth',1);
    line(app.UIAxes, [ancho ancho],[largo largo],[0 alto],'Color','black','LineWidth',1);
    line(app.UIAxes, [0 0],[largo largo],[0 alto],'Color','black','LineWidth',1);

    line(app.UIAxes, [0 ancho],[0 0],[alto alto],'Color','black','LineWidth',1);
    line(app.UIAxes, [ancho ancho],[0 largo],[alto alto],'Color','black','LineWidth',1);
    line(app.UIAxes, [ancho 0],[largo largo],[alto alto],'Color','black','LineWidth',1);
    line(app.UIAxes, [0 0],[largo 0],[alto alto],'Color','black','LineWidth',1);

    % 3. Vértices base en coordenadas homogéneas (4x8)
    V_orig = [
        0, ancho, ancho,     0,     0, ancho, ancho,     0; ...
        0,     0, largo, largo,     0,     0, largo, largo; ...
        0,     0,     0,     0,  alto,  alto,  alto,  alto; ...
        1,     1,     1,     1,     1,     1,     1,     1  ...
    ];

    % 4. Leer traslaciones y rotaciones ingresadas (convertir grados a rad)
    tx = app.TxEditField.Value;
    ty = app.TyEditField.Value;
    tz = app.TzEditField.Value;

    radX = deg2rad(app.RxEditField.Value);
    radY = deg2rad(app.RyEditField.Value);
    radZ = deg2rad(app.RzEditField.Value);

    % 5. Matrices de Transformación Homogénea (4x4)
    T = [1 0 0 tx; 
         0 1 0 ty; 
         0 0 1 tz; 
         0 0 0 1];

    Rx = [1      0          0     0;
          0  cos(radX) -sin(radX) 0;
          0  sin(radX)  cos(radX) 0;
          0      0          0     1];

    Ry = [ cos(radY) 0  sin(radY) 0;
               0     1      0     0;
          -sin(radY) 0  cos(radY) 0;
               0     0      0     1];

    Rz = [cos(radZ) -sin(radZ) 0 0;
          sin(radZ)  cos(radZ) 0 0;
              0          0     1 0;
              0          0     0 1];

    % Matriz total
    M = T * Rz * Ry * Rx;

    % 6. Animación del movimiento
    pasos = 30;
    h_mov = [];
    for k = 1:pasos
        alpha = k / pasos;

        radX_k = radX * alpha;
        radY_k = radY * alpha;
        radZ_k = radZ * alpha;
        tx_k   = tx * alpha;
        ty_k   = ty * alpha;
        tz_k   = tz * alpha;

        Tk  = [1 0 0 tx_k; 0 1 0 ty_k; 0 0 1 tz_k; 0 0 0 1];
        Rxk = [1 0 0 0; 0 cos(radX_k) -sin(radX_k) 0; 0 sin(radX_k) cos(radX_k) 0; 0 0 0 1];
        Ryk = [cos(radY_k) 0 sin(radY_k) 0; 0 1 0 0; -sin(radY_k) 0 cos(radY_k) 0; 0 0 0 1];
        Rzk = [cos(radZ_k) -sin(radZ_k) 0 0; sin(radZ_k) cos(radZ_k) 0 0; 0 0 1 0; 0 0 0 1];

        Mk = Tk * Rzk * Ryk * Rxk;
        V = Mk * V_orig;

        if ~isempty(h_mov)
            delete(h_mov);
        end

        % Dibujar cubo azul en cada frame
        h_mov(1)  = line(app.UIAxes, [V(1,1) V(1,2)], [V(2,1) V(2,2)], [V(3,1) V(3,2)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(2)  = line(app.UIAxes, [V(1,2) V(1,3)], [V(2,2) V(2,3)], [V(3,2) V(3,3)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(3)  = line(app.UIAxes, [V(1,3) V(1,4)], [V(2,3) V(2,4)], [V(3,3) V(3,4)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(4)  = line(app.UIAxes, [V(1,4) V(1,1)], [V(2,4) V(2,1)], [V(3,4) V(3,1)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(5)  = line(app.UIAxes, [V(1,1) V(1,5)], [V(2,1) V(2,5)], [V(3,1) V(3,5)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(6)  = line(app.UIAxes, [V(1,2) V(1,6)], [V(2,2) V(2,6)], [V(3,2) V(3,6)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(7)  = line(app.UIAxes, [V(1,3) V(1,7)], [V(2,3) V(2,7)], [V(3,3) V(3,7)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(8)  = line(app.UIAxes, [V(1,4) V(1,8)], [V(2,4) V(2,8)], [V(3,4) V(3,8)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(9)  = line(app.UIAxes, [V(1,5) V(1,6)], [V(2,5) V(2,6)], [V(3,5) V(3,6)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(10) = line(app.UIAxes, [V(1,6) V(1,7)], [V(2,6) V(2,7)], [V(3,6) V(3,7)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(11) = line(app.UIAxes, [V(1,7) V(1,8)], [V(2,7) V(2,8)], [V(3,7) V(3,8)], 'Color', 'blue', 'LineWidth', 2);
        h_mov(12) = line(app.UIAxes, [V(1,8) V(1,5)], [V(2,8) V(2,5)], [V(3,8) V(3,5)], 'Color', 'blue', 'LineWidth', 2);

        drawnow;
        pause(0.01);
    end

    % 7. Mostrar la ecuación y matriz resultante (si tienes un TextArea configurado)
    if isprop(app, 'EcuacionTextArea')
        txt = sprintf([ ...
            "ECUACIÓN UTILIZADA:\n" ...
            "P_final = T(Tx,Ty,Tz) * Rz(θz) * Ry(θy) * Rx(θx) * P_inicial\n\n" ...
            "MATRIZ RESULTANTE M (4x4):\n" ...
            "| %6.2f  %6.2f  %6.2f  %6.2f |\n" ...
            "| %6.2f  %6.2f  %6.2f  %6.2f |\n" ...
            "| %6.2f  %6.2f  %6.2f  %6.2f |\n" ...
            "| %6.2f  %6.2f  %6.2f  %6.2f |\n"], ...
            M(1,:), M(2,:), M(3,:), M(4,:));
        app.EcuacionTextArea.Value = txt;
    
end
        end
    end

    % Component initialization
    methods (Access = private)

        % Create UIFigure and components
        function createComponents(app)

            % Create UIFigure and hide until all components are created
            app.UIFigure = uifigure('Visible', 'off');
            app.UIFigure.Position = [100 100 640 480];
            app.UIFigure.Name = 'MATLAB App';

            % Create UIAxes
            app.UIAxes = uiaxes(app.UIFigure);
            title(app.UIAxes, 'Title')
            xlabel(app.UIAxes, 'X')
            ylabel(app.UIAxes, 'Y')
            zlabel(app.UIAxes, 'Z')
            app.UIAxes.Position = [221 45 300 185];

            % Create TxEditFieldLabel
            app.TxEditFieldLabel = uilabel(app.UIFigure);
            app.TxEditFieldLabel.HorizontalAlignment = 'right';
            app.TxEditFieldLabel.Position = [109 376 25 22];
            app.TxEditFieldLabel.Text = 'Tx';

            % Create TxEditField
            app.TxEditField = uieditfield(app.UIFigure, 'numeric');
            app.TxEditField.Position = [149 376 100 22];

            % Create TyEditFieldLabel
            app.TyEditFieldLabel = uilabel(app.UIFigure);
            app.TyEditFieldLabel.HorizontalAlignment = 'right';
            app.TyEditFieldLabel.Position = [116 327 25 22];
            app.TyEditFieldLabel.Text = 'Ty';

            % Create TyEditField
            app.TyEditField = uieditfield(app.UIFigure, 'numeric');
            app.TyEditField.Position = [156 327 100 22];

            % Create TzEditFieldLabel
            app.TzEditFieldLabel = uilabel(app.UIFigure);
            app.TzEditFieldLabel.HorizontalAlignment = 'right';
            app.TzEditFieldLabel.Position = [116 278 25 22];
            app.TzEditFieldLabel.Text = 'Tz';

            % Create TzEditField
            app.TzEditField = uieditfield(app.UIFigure, 'numeric');
            app.TzEditField.Position = [156 278 100 22];

            % Create RxEditFieldLabel
            app.RxEditFieldLabel = uilabel(app.UIFigure);
            app.RxEditFieldLabel.HorizontalAlignment = 'right';
            app.RxEditFieldLabel.Position = [423 376 25 22];
            app.RxEditFieldLabel.Text = 'Rx';

            % Create RxEditField
            app.RxEditField = uieditfield(app.UIFigure, 'numeric');
            app.RxEditField.Position = [463 376 100 22];

            % Create RyEditFieldLabel
            app.RyEditFieldLabel = uilabel(app.UIFigure);
            app.RyEditFieldLabel.HorizontalAlignment = 'right';
            app.RyEditFieldLabel.Position = [423 327 25 22];
            app.RyEditFieldLabel.Text = 'Ry';

            % Create RyEditField
            app.RyEditField = uieditfield(app.UIFigure, 'numeric');
            app.RyEditField.Position = [463 327 100 22];

            % Create RzEditFieldLabel
            app.RzEditFieldLabel = uilabel(app.UIFigure);
            app.RzEditFieldLabel.HorizontalAlignment = 'right';
            app.RzEditFieldLabel.Position = [423 278 25 22];
            app.RzEditFieldLabel.Text = 'Rz';

            % Create RzEditField
            app.RzEditField = uieditfield(app.UIFigure, 'numeric');
            app.RzEditField.Position = [463 278 100 22];

            % Create TraslacionLabel
            app.TraslacionLabel = uilabel(app.UIFigure);
            app.TraslacionLabel.Position = [156 419 60 22];
            app.TraslacionLabel.Text = 'Traslacion';

            % Create RotacionLabel
            app.RotacionLabel = uilabel(app.UIFigure);
            app.RotacionLabel.Position = [480 419 52 22];
            app.RotacionLabel.Text = 'Rotacion';

            % Create botonButton
            app.botonButton = uibutton(app.UIFigure, 'push');
            app.botonButton.ButtonPushedFcn = createCallbackFcn(app, @botonButtonPushed, true);
            app.botonButton.Position = [72 229 100 23];
            app.botonButton.Text = 'boton';

            % Show the figure after all components are created
            app.UIFigure.Visible = 'on';
        end
    end

    % App creation and deletion
    methods (Access = public)

        % Construct app
        function app = app1

            % Create UIFigure and components
            createComponents(app)

            % Register the app with App Designer
            registerApp(app, app.UIFigure)

            if nargout == 0
                clear app
            end
        end

        % Code that executes before app deletion
        function delete(app)

            % Delete UIFigure when app is deleted
            delete(app.UIFigure)
        end
    end
end
